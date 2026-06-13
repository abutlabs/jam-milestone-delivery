# JAM Milestone Delivery

## Company/Team details

- Company/Team's name: **abutlabs**
- Company/Team's GitHub: https://github.com/abutlabs
- Learning Lasair (Developers Education): https://lasair.netlify.app 
- Programming language and language set: **OCaml** (Set D — "Correct code")
- Link/s to previous delivery/ies: — (first delivery)

## Documentation checklist

We declare that:

- [x] we have completed **the Web3 Foundation KYC/KYB process**.
- [x] we used **a clear and permissive open-source license** — MIT (`LICENSE` in the repository root).
- [x] we submitted **code developed in private, with commit hashes placed, in a timely fashion, on a major public blockchain**.
  - Development began 2026-01-02; continuous commit history since.
  - On-chain timestamp via `system.remarkWithEvent` on **Polkadot Asset Hub** (transitively timestamps the whole reachable history, as each commit names its parents):
    - Commit attested: `f7a90db2c588905b428dbee1f59d63a4b2c4aecc`
    - Account: `15AWQjAZ9Ev9uhcYJdfwQzXA2VRDn2oLgZTkBzRRT7sZNDgs`
    - Block: `0x31c079cf09f3a8eb29c816b1d2a8d70fc2f451691451abbbb9acfa42d0b9b361`
    - Extrinsic: `0xfc26626aaedcbaad107e502f26b339e63c16f9a9b28ab65b979ea0b8cba2d6fc`
    - Date: 2026-06-13, finalized.
    - Verify: https://assethub-polkadot.subscan.io/extrinsic/0xfc26626aaedcbaad107e502f26b339e63c16f9a9b28ab65b979ea0b8cba2d6fc
- [x] we used third party libraries for: **cryptographic primitives** (erasure-coding, Bandersnatch Ring VRF, Ed25519).
  - Bandersnatch / Ring + IETF VRF: Rust [`w3f/ring-vrf` / ark-vrf](https://github.com/w3f/ring-vrf) via C FFI (`rust/bandersnatch-ffi`).
  - Ed25519 signature verification: Rust FFI (`rust/ed25519-ffi`).
  - Erasure coding (Reed–Solomon): Rust FFI (`rust/erasure-ffi`).
  - blake2b / keccak: OCaml [`digestif`](https://github.com/mirage/digestif).
  - The JAM codec, PVM, and all state-transition logic are our own OCaml.
- [x] we provided **Gas, trie/DB, signature-verification performance tests** to be run on standard hardware.
  - Gas: exact PVM gas accounting, verified bit-for-bit against the reference polkavm interpreter (`tools/pvm-replay`); per-block import latency is measurable with `dune exec bin/fuzz_driver.exe -- --timing --traces <family>` (comfortably sub-second on tiny spec). Formal availability (EC/DB) benchmarking is scheduled for the performance milestones (M3/M4).
- [x] we viewed the following **JAM implementation code** before / during our implementation (full record: `docs/DISCLOSURES.md`):
  - **polkavm** (Parity, Rust): semantic reference for our independently written OCaml PVM; we built a record-replay harness (`tools/pvm-replay`) that re-executes our recorded invocations through real polkavm and diffs PC + registers to prove instruction-level equivalence.
  - **typeberry** (FluffyLabs, TypeScript): black-box conformance oracle (host-call IO tracing) and source consultation to resolve Graypaper v0.7.2 ambiguities (accumulation sequencing, fetch context, host-call gas, work-result discriminants, chain-spec constants).
  - **turbojam** (C++): read `machine.cpp` for heap/sbrk semantics.
  - **jam-pvm-sdk / jam-types** (Rust crates): read for the exact `ProtocolParameters` layout the conformance test services are compiled against.
  - In all cases we extracted protocol facts to debug conformance; no code was translated or copied. The implementation is original OCaml.
- [x] we have **not** had private conversations with **other implementers**.
- [x] we have **not** had **concerns about collusion**.
- [x] we agree to a recorded interview by the *Polkadot Technical Fellowship* on any matter arising from this milestone submission.
- [x] we understand that this milestone submission will need to be ratified with an on-chain remark by the *Polkadot Technical Fellowship* before it can be merged.

## Context

Lasair is an OCaml implementation of the JAM protocol (Graypaper v0.7.2),
developed by abutlabs since January 2026. It is, to our knowledge, the only
active OCaml (Set D) JAM client.

This milestone delivers **M1 (IMPORTER)** on the Validating Node Path:
state-transitioning conformance tests pass and the client can import blocks,
with its own PVM interpreter (no reused VM).

Evidence, all reproducible in-repo:

- **Registered conformance suite** — all nine STF component suites, codec,
  trie, shuffle and erasure vectors, and **all eight** block-import trace
  families (fallback, safrole, storage, storage_light, preimages,
  preimages_light, fuzzy, fuzzy_light) — 1,000 trace vectors, with exact PVM
  gas accounting verified bit-for-bit against polkavm. The full gate
  (`scripts/conformance.sh`) runs 24 checks.
- **Fuzzer protocol (`fuzz-proto`)** — our self-built driver replays every
  public source over the Unix-socket interface, all green:
  - recorded `forks` / `no_forks` sessions: **204/204**, byte-exact;
  - the official `minifuzz` harness on the recorded session: **102/102**, 0
    mismatches;
  - all eight families pushed through the socket as binary, chained from
    genesis: **1,000/1,000**;
  - the 205 fuzz-report failure seeds from other teams' fuzzing runs:
    **746/746**.
- **Adversarial hardening** — a mutation engine corrupts known-valid blocks
  and asserts the trilogy (never crash, never hang, reject without state
  change); a 2-hour soak fired **271,320 mutations** across all families
  with 0 crash / 0 hang / 0 wrong-accept (`docs/ADVERSARIAL_AUDIT.md`).
- **Performance** — comfortably sub-second per `ImportBlock` on tiny spec
  (mean ~40 ms, p99 ~0.4 s).

## Deliverables

- [x] 1. Validating Node Path
- [ ] 2. Non-PVM Validating Node Path
- [ ] 3. Light Node Path

- **Milestone: 1 (IMPORTER)**

| Number | Deliverable | Link | Notes |
|-------:|-------------|------|-------|
| 0a. | License | `LICENSE` @ `f7a90db` | MIT |
| 0b. | Documentation | `README.md`, `docs/` @ `f7a90db` | build/run, debugging methodology, adversarial audit; plus `learning-lasair` (Conformance Lab + Mutation Lab) |
| 0c. | Testing guide | `README.md` + `scripts/conformance.sh` @ `f7a90db` | full suite + codec oracle + mutation smoke (24 checks) |
| 0d. | Docker | `Dockerfile` + `deliveries/lasair/RUNNING.md` @ `f7a90db` | builds `lasair:0.7.2`; ENTRYPOINT binds `--socket {TARGET_SOCK}`, compatible with jam-conformance `scripts/target.py` (`deliveries/lasair/targets-lasair.json`) |
| 0e. | Published target image | [`ghcr.io/abutlabs/lasair:0.7.2`](https://github.com/users/abutlabs/packages/container/package/lasair) | public GHCR image, CI-built for `linux/amd64` (`.github/workflows/publish-target.yml`), digest `sha256:3f18badf…f9a534`; `docker pull ghcr.io/abutlabs/lasair:0.7.2`, then `target.py get lasair` resolves it via `targets-lasair.json` |
| 1. | Source code (JAM client) | https://github.com/abutlabs/lasair @ `f7a90db2c588905b428dbee1f59d63a4b2c4aecc` | OCaml JAM client, conformance harness, PVM replay tooling, mutation engine |
| 2. | Fuzz target | `bin/conformance_target.ml` @ `f7a90db` | fuzz-proto v1 socket interface |
| 3. | Conformance evidence | `scripts/conformance.sh` (24/24), `bin/fuzz_driver.exe` (1,204/1,204), `minifuzz` (102/102), `--mutate` (271k mutations, 0 fail) @ `f7a90db` | reproducible in-repo; `docs/ADVERSARIAL_AUDIT.md` |

## Running the published target (no repository access required)

Conformance can be run against the public image without access to the
source repository:

```bash
# 1. pull (linux/amd64, anonymous — no login required)
docker pull ghcr.io/abutlabs/lasair:0.7.2

# 2. register it in jam-conformance scripts/targets.json:
#      "lasair": { "image": "ghcr.io/abutlabs/lasair:0.7.2",
#                  "cmd": "--socket {TARGET_SOCK}", "gp_version": "0.7.2" }

# 3. launch; the fuzzer connects over the Unix socket
python3 scripts/target.py get lasair
python3 scripts/target.py run lasair
```

The image ENTRYPOINT is the target binary; `target.py` appends
`--socket {TARGET_SOCK}`. Pinned digest
`sha256:3f18badf11588bc66162979819a56319df40997030ea2c331b4d977ac9f9a534`.

## Additional Information

A debugging methodology we believe is of general interest to implementers:
a record-replay harness (`tools/pvm-replay`) captures every host-call effect
from a lasair run and re-executes the invocation inside the reference
polkavm interpreter, proving instruction-level equivalence and cleanly
separating PVM bugs from host-layer bugs. It took our storage trace family
from 6/101 to 101/101 in one day. The mutation engine and its interactive
"Mutation Lab" teaching page extend the same forensic approach to
adversarial input.

I used generative AI throughout this project, details emailed to jam@web3.foundation 2026-06-13.