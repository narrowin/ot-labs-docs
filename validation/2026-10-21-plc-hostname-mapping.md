# PLC list and process image in the exercise introduction

The exercise opening uses a plain one-to-one list and the user's supplied water-treatment visualization.

- **PLC1**: `abb-800xa-vlan40`
- **PLC2A**: `wago-plc2a-vlan10`
- **PLC2B**: `wago-plc2b-vlan10`
- **PLC3**: `ctrlx-plc3-vlan20`
- **PLC4**: `schneider-plc4-vlan30`

## Placement

The same five-item list and image appear near the beginning of English and German Exercise 1 and both exercise overviews. Exercise 1 includes the reference because the in-app reader opens the seven numbered lessons directly.

The shared image is `docs/img/water-treatment-overview.png`. Both language trees already link `img` to that shared directory. Markdown uses `../img/water-treatment-overview.png`; the existing reader resolves its filename through the course asset endpoint. No frontend or backend change is needed.

PLC2A and PLC2B have separate entries. The introduction does not assign them a shared node, invent output ownership or present a new diagram. The supplied visualization is copied unchanged.

## Verification

- All five hostname mappings match the checked-in lab topology and the handout's controller allocation.
- The image is 1365 by 1284 pixels, 419,093 bytes. Its SHA-256 matches the attachment: `6a22c94f920287f2200e8af831bf6caeae01a39e16c6598328066a538e649415`.
- English and German strict MkDocs builds pass.
- The existing `read_exercise` and `exercise_asset` functions successfully resolve Exercise 1 and the shared image for both languages using the local course directory.
- Rendered English and German introductions were inspected. The list and image load within the content column.
- Exercise commands and packet filters are unchanged. Separate German proofreading edits remain outside this change.

## Limits

The picture records an example process state; its alarms, communication indicators and actuator states are not acceptance criteria for the networking lessons. The list and image are now available in English and German through the [live reader publication](2026-10-21-live-reader-publication.md).
