# Copilot instructions

## Build and verification

See `INSTALL.md` for toolchain setup. The default `make` target builds the original-compiler FireRed ROM (`pokefirered.gba`); this path requires `agbcc`. Useful variants and verification commands:

```sh
make -j"$(nproc)" firered                 # FireRed, revision 0
make leafgreen                            # LeafGreen, revision 0
make firered_rev1                         # FireRed, revision 1
make firered_switch                       # FireRed, revision 10
make compare_firered                      # Build one target and check its reference SHA-1
make compare_leafgreen
make compare_firered_rev1
make compare_firered_switch
make modern                               # Build with arm-none-eabi-gcc instead of agbcc
```

There is no standalone unit-test or lint target in the repository. `compare_<variant>` is the narrowest built-in verification: it compares the ROM against its checked-in SHA-1 reference. Intentional gameplay or data changes will naturally change the checksum. For regular builds and more variants, see `INSTALL.md` and the targets in `Makefile`.

## Architecture

This is a decompilation of the English Game Boy Advance FireRed and LeafGreen games. The Makefiles build six game/revision combinations from C, ARM assembly, event-script assembly, and source assets. Game version and revision are selected through Make targets and preprocessor definitions; the default original-compiler build uses `agbcc`, while `MODERN=1` uses `arm-none-eabi-gcc`.

Gameplay and engine code lives primarily in `src/`, with interfaces in `include/`. `constants/` and `include/constants/` define shared IDs and flags, while `data/` contains structured game data and event scripts. Map source is split across `data/maps/<Map>/map.json`, `scripts.inc`, and `text.inc`; layouts are under `data/layouts/`. Map JSON and layouts are processed by `mapjson` into assembly includes and constants during the build. Edit the source JSON/scripts/text rather than generated map includes.

Assets under `graphics/` and `sound/` are converted by the asset rules (`graphics_file_rules.mk`, `spritesheet_rules.mk`, `tileset_rules.mk`, and `audio_rules.mk`) using programs in `tools/`. JSON data rules also generate headers consumed by C. The build preprocesses C and assembly with the repository’s custom `preproc` and `charmap.txt`, then assembles and links objects using the revision-specific linker scripts and symbol maps. These scripts, memory-section declarations, and symbol data preserve the original ROM layout; changes to them can affect binary matching.

## Repository-specific conventions

- Keep changes in the source files consumed by the Make rules. Generated map includes and generated data headers are build outputs and can be recreated.
- C code relies on project headers and memory-placement macros such as `IWRAM_DATA`, `EWRAM_DATA`, and `COMMON_DATA`. These correspond to hardware memory regions and linker sections; preserve the appropriate section when changing or adding globals.
- Game text is encoded through `charmap.txt` and the custom preprocessing pipeline, not as ordinary UTF-8 output. Follow neighboring `.string` entries for control codes, placeholders, and line breaks.
- Event scripts are assembler-like DSLs with project macros and labels, conventionally named with the map prefix (for example `PalletTown_EventScript_*`). Map event/object references in JSON must agree with script labels and shared constants.
- Original-ROM matching is variant-specific. Use the corresponding `compare_*` target when validating a baseline; the modern compiler target is for building and is not the checksum-matching path.

## Safety rules

- Inspect the relevant files and surrounding context before editing.
- Explain the planned change before making it, especially when it affects build, data, or ROM layout.
- Stay within the approved scope; do not broaden the task or fix unrelated issues.
- Avoid destructive actions: do not delete files, overwrite generated outputs without reason, or make risky changes without justification.
- Never commit without my explicit approval.
- After implementation, always show the Git diff and summarize all changed files.
- Do not commit.
