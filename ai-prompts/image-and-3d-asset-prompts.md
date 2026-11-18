# Image and 3D Asset Prompts

> Agent-facing guide for generating 2D images (icons, thumbnails, textures, UI art) and 3D assets via Blender Python (`bpy`) for Roblox games.
> Related: `roblox-brain-local/skills/gameplay/roblox-gui/`, `roblox-building/`, `roblox-animation-vfx/`.

---

## Part A — Image generation prompts

### A.1 Prompt formula

```text
[Subject], [style], [composition/camera], [lighting], [color palette], [background], [technical constraints]
```

Always specify:
- **Style** explicitly (e.g. "stylized low-poly Roblox-style 3D render", "flat vector icon", "hand-painted texture").
- **Background** (`transparent background`, `solid #FFFFFF`, or a scene).
- **Aspect ratio / resolution** matching the target Roblox slot (table A.3).
- **Negative constraints**: no text, no watermark, no logos, no real brand marks, no real people.

### A.2 Templates

**Game icon (512×512):**
```text
A bold, colorful game icon for a Roblox [GENRE] game featuring [MAIN SUBJECT] in a dynamic pose,
stylized 3D cartoon render, centered composition, subject fills 70% of frame, strong rim lighting,
saturated [PRIMARY COLOR] and [ACCENT COLOR] palette, simple gradient background,
high contrast, readable at small sizes, no text, no watermark, square 1:1.
```

**Game thumbnail (1920×1080):**
```text
Wide cinematic scene from a Roblox [GENRE] game: [ACTION DESCRIPTION], blocky stylized characters,
dramatic three-quarter camera angle, depth of field, golden-hour lighting, vibrant colors,
clear focal point on the left third, empty space on the right for title overlay,
no text, no watermark, 16:9.
```

**UI button / inventory icon (256×256, transparent):**
```text
Single [ITEM] icon, clean stylized game-UI style, slight 3/4 view, soft top-left light,
thick dark outline, flat cel shading, transparent background, centered, no text, 1:1.
```

**Tileable texture (1024×1024):**
```text
Seamless tileable [MATERIAL] texture, top-down orthographic, even flat lighting, no shadows,
no perspective, no objects, stylized hand-painted look, consistent scale, 1:1.
```

**Skybox face:**
```text
[ENVIRONMENT] sky, stylized, soft gradient, scattered clouds, no ground, no sun disc in frame,
seamless edges, square 1:1.
```

### A.3 Roblox image targets

| Asset | Recommended size | Notes |
| :-- | :-- | :-- |
| Game icon | 512 × 512 | Square. Avoid text; it is unreadable at small sizes. |
| Game thumbnail | 1920 × 1080 | 16:9. Keep focal content in the center safe area. |
| Decal / ImageLabel | ≤ 1024 × 1024 | Power-of-two sizes compress best. |
| Texture / SurfaceAppearance maps | 1024 × 1024 (max 4096 for SurfaceAppearance) | Color, Normal, Metalness, Roughness maps. |
| UI icons | 128–256 square | Transparent PNG. Consistent padding across a set. |

Moderation: uploaded images go through Roblox moderation. Never request content that imitates real brands, real people, copyrighted characters, or violates Roblox Community Standards.

---

## Part B — Blender `bpy` scripting for Roblox assets

### B.1 Hard rules for generated `bpy` scripts

1. **Target Blender 3.6 LTS / 4.x API.** Read `bpy.app.version` and branch if needed.
2. **Never hardcode localized node names.** Find nodes by type:
   `next(n for n in mat.node_tree.nodes if n.type == "BSDF_PRINCIPLED")`.
3. **Never hardcode enum identifiers blindly**; read valid values from `bl_rna` when unsure.
4. **Set material color on shader node inputs**, not `material.diffuse_color` (viewport only).
5. **Scale for Roblox:** 1 Blender unit = 1 stud when exporting FBX with scale 1.0 and Roblox import set to studs. State the convention used in a header comment.
6. **Apply transforms** (`bpy.ops.object.transform_apply(location=False, rotation=True, scale=True)`) before export.
7. **Polygon budget:** Roblox MeshPart limit is 20,000 triangles per mesh (verify current limit in Creator Docs). Target ≤ 5,000 for props, ≤ 10,000 for hero assets.
8. **Origin** at the object's bottom center for props so it sits on the ground in Studio.
9. **Clean names:** name objects, meshes and materials meaningfully; no `Cube.001`.
10. **Do not delete the user's existing objects.** Create new objects in a dedicated collection.
11. **Export** to FBX (or OBJ) with `-Y forward`, `Z up` unless the user's pipeline says otherwise.

### B.2 Reference script — stylized low-poly prop

```python
# Blender bpy script: stylized low-poly crate for Roblox
# Convention: 1 Blender unit = 1 stud. Origin = bottom center.
import bpy
import bmesh

COLLECTION_NAME = "RobloxAssets"
ASSET_NAME = "WoodCrate"
SIZE = 4.0  # studs

def get_collection(name):
    col = bpy.data.collections.get(name)
    if col is None:
        col = bpy.data.collections.new(name)
        bpy.context.scene.collection.children.link(col)
    return col

def make_material(name, rgba, roughness=0.8):
    mat = bpy.data.materials.get(name) or bpy.data.materials.new(name)
    mat.use_nodes = True
    bsdf = next(n for n in mat.node_tree.nodes if n.type == "BSDF_PRINCIPLED")
    bsdf.inputs["Base Color"].default_value = rgba
    bsdf.inputs["Roughness"].default_value = roughness
    return mat

col = get_collection(COLLECTION_NAME)

mesh = bpy.data.meshes.new(ASSET_NAME + "_Mesh")
obj = bpy.data.objects.new(ASSET_NAME, mesh)
col.objects.link(obj)

bm = bmesh.new()
bmesh.ops.create_cube(bm, size=SIZE)
# Move so origin is at bottom center
bmesh.ops.translate(bm, verts=bm.verts, vec=(0, 0, SIZE / 2))
# Bevel edges for a stylized look (keeps poly count low)
bmesh.ops.bevel(bm, geom=bm.edges[:], offset=0.15, segments=1, affect="EDGES")
bm.to_mesh(mesh)
bm.free()

obj.data.materials.append(make_material("Wood_Mat", (0.55, 0.35, 0.18, 1.0)))

tri_count = sum(len(p.vertices) - 2 for p in mesh.polygons)
print(f"[{ASSET_NAME}] triangles: {tri_count}")
assert tri_count <= 5000, "Over prop triangle budget"
```

### B.3 Reference script — FBX export for Roblox

```python
import bpy, os

EXPORT_DIR = bpy.path.abspath("//exports")  # next to the .blend file
os.makedirs(EXPORT_DIR, exist_ok=True)

obj = bpy.data.objects["WoodCrate"]
bpy.ops.object.select_all(action="DESELECT")
obj.select_set(True)
bpy.context.view_layer.objects.active = obj
bpy.ops.object.transform_apply(location=False, rotation=True, scale=True)

bpy.ops.export_scene.fbx(
    filepath=os.path.join(EXPORT_DIR, obj.name + ".fbx"),
    use_selection=True,
    apply_scale_options="FBX_SCALE_UNITS",
    axis_forward="-Y",
    axis_up="Z",
    mesh_smooth_type="FACE",
    add_leaf_bones=False,
    bake_anim=False,
)
print("Exported", obj.name)
```

### B.4 `bpy` prompt template for AI agents

```text
Write a Blender bpy script (Blender 4.x) that creates a [ASSET] for a Roblox game.
Constraints:
- 1 Blender unit = 1 stud; overall size about [W x D x H] studs.
- Stylized low-poly, under [N] triangles. Print the triangle count.
- Origin at bottom center. Apply rotation and scale.
- Put everything in a collection named "RobloxAssets". Do not touch other objects.
- Materials via Principled BSDF found by node type, colors: [PALETTE].
- Meaningful names for objects, meshes, materials.
- Optional: export to FBX in //exports with axis_forward="-Y", axis_up="Z".
```

### B.5 Asset QA checklist

- [ ] Correct scale in Studio (compare against a 5-stud R15 character).
- [ ] Triangle count within budget.
- [ ] Origin/pivot sits on the ground.
- [ ] No inverted normals (check in Studio with a light source).
- [ ] `CollisionFidelity` set appropriately after import (`Box`/`Hull` for props).
- [ ] Textures within size limits; `SurfaceAppearance` maps assigned if PBR.
- [ ] Names are clean; no leftover `Cube.001` / `Material.002`.
