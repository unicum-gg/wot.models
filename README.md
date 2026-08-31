# wot.models

**World of Tanks vehicle geometry**, the armor a shell has to get through and the
model a player sees, one branch per client build. Extracted straight from the
update CDN by [`unicum-gg/wot.build`](https://github.com/unicum-gg/wot.build),
with no game client installed. Used by [unicum.gg](https://unicum.gg) to draw a
tank and to show where its plates are.

This is the piece the other mirrors leave out. `wot.src` and its kin publish a
vehicle's scripts, its XML and its armor *values*, but not its meshes, because
those are binary, and that gap is what stops a site from drawing a tank. It is
meant as a shared community resource in the same spirit, so it covers every
publisher rather than only what unicum.gg happens to use. Reuse and
contributions welcome.

## Branches

| Branch | Client | Update service | guid |
| --- | --- | --- | --- |
| [`WG`](../../tree/WG) | Wargaming release | `wgus-woteu.wargaming.net` | `WOT.EU.PRODUCTION` |
| [`WG_CT`](../../tree/WG_CT) | Wargaming Common Test | `wgus-wotct.wargaming.net` | `WOT.CT.PRODUCTION` |
| [`Lesta`](../../tree/Lesta) | Lesta release (Мир танков) | `lstus-ru.lesta.ru` | `MT.RU.PRODUCTION` |
| [`Lesta_PT`](../../tree/Lesta_PT) | Lesta public test | `lstus-ru.lesta.ru` | `MT.PT.PRODUCTION` |

One copy of the release branch serves EU, NA, ASIA and CN: the geometry is the
same everywhere, unlike the sources, which differ per region. A vehicle appears
on the Common Test branch weeks before it is released, which is what that one is
for.

## What is published

```
vehicles/<nation>/<code>/
    collision.json      armor geometry, one entry per piece
    model.json          how the pieces fit together, and what they are drawn with
    Hull.glb            one file per piece: hull, each turret, each gun, the chassis
    Turret_01.glb
    <texture>.webp      the vehicle's own textures
vehicles/<nation>/tracks/<texture>.webp     textures a nation shares between vehicles
```

### `collision.json`

The armor. Each piece is one mesh, split into groups named after the plates the vehicle's
own armor table lists, so a thickness is joined by name:

```json
{ "parts": { "Hull": {
    "positions": [-0.1244, 0.6316, 2.5906, ...],
    "indices": [0, 1, 2, ...],
    "groups": [{ "name": "armor_16", "start": 0, "count": 6 }, ...]
} } }
```

`armor_N` matches `<armor_N>` in the vehicle's XML. The other names are the parts the game
treats specially: `leftTrack`, `rightTrack`, `gun`, `surveyingDevice`.

Thicknesses are deliberately **not** included. They change with balance patches while the
geometry does not, and they are already published, in full, by the script mirrors. Joining
them here would make this repository a second source of truth for a number it does not own.

### `model.json`

The visual model. `pieces` says which `.glb` to load and where it attaches, `materials`
says what to draw it with:

```json
{
  "pieces": {
    "Turret_01": {
      "glb": "Turret_01.glb",
      "hardpoints": { "HP_gunJoint": [0, 0.349, 0.693] },
      "meshes": [{ "name": "vertices", "materials": [3] }]
    }
  },
  "materials": [{
    "name": "track_mat_R_skinned",
    "shader": "shaders/std_effects/PBS_tank_uvtransform_skinned_ao.fx",
    "textures": { "diffuseMap": { "path": "vehicles/…/IS_7_track_AM.webp", "colorSpace": "srgb" } },
    "values": { "g_detailUVTiling": [4.2, 1.05, 0, 0], "alphaReference": 64 },
    "doubleSided": true,
    "alphaTest": 0.251
  }]
}
```

A few things are worth knowing before drawing this:

- **Pieces are modelled around their own origin.** A hull declares `HP_turretJoint`, a
  turret declares `HP_gunJoint`, and a vehicle with two turrets numbers them. Drop a piece
  in at the origin and it ends up inside the one that should carry it.
- **`doubleSided` and `alphaTest` are load-bearing.** They are what cuts the gaps out of a
  track and keeps its far side from vanishing. Ignore them and a track draws as a solid
  ribbon.
- **Some meshes carry a second UV set** (`TEXCOORD_1`). Shaders whose name ends in `_ao`
  sample occlusion with it, because the first set is tiled many times over.
- Geometry is **Y-up**, the glTF convention, and needs no correction.
- A material only lists textures that were actually published. The client references a
  couple that it no longer ships, and those are dropped rather than left as dead links.

### Textures

Published as WebP, rebuilt for what each one actually is rather than copied channel for
channel:

| Client suffix | What it is | Published as |
| --- | --- | --- |
| `AM` | base colour | RGB, sRGB (plus alpha where a material cuts it out) |
| `ANM` | normals, stored in two channels | a full tangent-space normal map |
| `GMM` | gloss, metalness, mask | glTF metal-roughness (roughness in green, metalness in blue) |
| `AO` | occlusion, one channel | greyscale |

## Notice

Assets provided in the repository are the property of their sole owners.
