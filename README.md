# Machinist E5-MR9A PRO V1.2 (Q87) firmware

A [Biosmod Engine](https://github.com/juliopolycarpo/biosmod-engine) catalog: the board profile, findings, patches and evidence for one board or one board family. `bme guide catalogs` describes every document.

## This board

| | |
|---|---|
| Board | Machinist E5-MR9A PRO V1.2, Intel Q87, Nuvoton NCT5532D Super I/O |
| Board profile | `boards/machinist-mr9a-pro-q87-rev-1.2.json` |
| Firmware build | `mnx99mr9apro-230322`: MNX99MR9APRO 2023-03-22, version string `X9R9P Ver:005 03/22/2023` |
| Stock image | `MNX99MR9APRO_230322_original_90DEDD1D.bin`, 16,777,216 bytes, SHA-256 `4036d26e7ff72160625fd8a1d9cc33095116d02342f7e5398f7fdb1bfb71b2a6` |

```sh
bme scan dump.bin --catalog . --allow-unsigned-local
bme patch dump.bin --recommended --catalog . --allow-unsigned-local
```

`bme` identifies the build by its modules and patches a module only when its hash matches the recorded source. Read `bme guide flashing` before you write an image to the chip.

## Shared with the PR9

The Machinist PR9 (B85) ships a build of the same X99-derived AMI code base, so most defects exist on both boards. A shared defect keeps one id in both repositories: `firmware_pr9-b85` records it with the same finding text and its own patch variant. A finding whose `affects.label` is `MR9A PRO and PR9` is shared. The catalog bundled with `bme` holds the records of both boards.

`bme` checks that the two repositories agree on every shared record, and fails with the record, the field and both values when they differ:

```sh
bme catalog merge . ../firmware_pr9-b85 --check
```

The catalog part of this repository equals what `bme catalog split official --board machinist-mr9a-pro-q87-rev-1.2` writes from the bundled catalog. A record added here reaches the bundled catalog through `bme catalog merge`.

## Layout

| Path | Holds | In git |
|---|---|---|
| `catalog.json` | the index: id, version, and the length and SHA-256 of every file in `boards/`, `findings/`, `patches/`, `evidence/` and `provenance/` | yes |
| `boards/` | board and revision profiles with the firmware builds they ship | yes |
| `findings/` | one defect or observation per file, patched or not | yes |
| `patches/` | one patch record per file, with one variant per firmware build | yes |
| `evidence/` | one hardware observation per file, bound to the image that was flashed | yes |
| `provenance/` | sources you wrote: assembly, harness scripts | yes |
| `notes/` | investigation notes in prose | yes |
| `work/` | dumps, cases, extracted modules, tool output | no |

Git records no empty directory: each directory reaches a clone with its first file.

## What stays out of git

`.gitignore` admits the paths marked "yes", `README*`, `LICENSE*`, `.github/`, `catalog.json.minisig`, `.gitignore` and `.gitattributes`. Anything else at the top level is ignored: a new top-level file does not appear in `git status` until you add a `!/name` line for it.

In every directory it also ignores firmware images and extracted modules (`*.bin`, `*.rom`, `*.pe`), ACPI tables, analysis projects, executables, archives, videos, PDFs and key files.

Patterns cannot read text. Before you commit a file under `notes/` or `provenance/`, check that it has no:

- serial number, UUID or MAC address (kernel logs, `dmidecode` output and network listings carry them);
- disassembly or decompiled code of a vendor module beyond the lines a finding quotes;
- path of your machine.

Name a dump or a flashed image by file name, SHA-256 and size. Never commit it.

## Work on it

```sh
cp /path/to/dump.bin work/
bme case init work/dump.bin -o work/case      # scan, draft a board profile and firmware build
bme case export work/case -o work/submission  # draft records to review and copy into boards/, findings/, evidence/, provenance/
bme catalog index .                           # after every change to a record
bme catalog validate .
bme scan work/dump.bin --catalog . --allow-unsigned-local
```

`bme guide cases` and `bme guide porting` cover the investigation itself.

## Commits

One record per commit, with the id in the scope and the defect in the subject, so `git log` and code search find it by id, module or symptom:

```text
feat(board): add ACME X99-A rev 1.0 with firmware X99A 2023-01-10
feat(H11): record unbounded PCU mailbox and UBOX semaphore waits
fix(B09): widen every inventoried SMM configuration-table counter
test(evidence): boot ACME X99-A with B01-B06 and B02-B03 flashed
docs(notes): describe the S3 resume investigation
```

`feat` adds a board or a finding, `fix` adds or changes a patch record together with the findings it fixes, `test` adds a hardware observation. A commit that changes a file in an indexed directory carries the re-indexed `catalog.json` and passes `bme catalog validate .`.

## Releases

`bme catalog update` refuses a version it already accepted with other content, and a version older than the newest one it accepted. So `version` in `catalog.json` changes only in a release commit: raise it, run `bme catalog index .`, sign `catalog.json` with minisign, commit `catalog.json.minisig` and tag the commit. Users register the tag's raw `catalog.json` URL:

```sh
bme catalog add mr9a-pro-rev1.2 https://example.org/path/to/catalog.json --trust-key <public key>
bme catalog update mr9a-pro-rev1.2
```

Give them the public key outside this repository. A key committed here proves nothing.
