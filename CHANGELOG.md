# Changelog

All notable changes to this project will be documented in this file.

This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## \[[0.9.0-dev.5](https://github.com/holochain/holochain-hdi/compare/v0.9.0-dev.4...v0.9.0-dev.5)\] - 2026-09-11

### Miscellaneous Tasks

- Upgrade schemars to v1.2 by @jost-s in [#12](https://github.com/holochain/holochain-hdi/pull/12)

### Automated Changes

- *(deps)* Bump syn from 2.0.119 to 3.0.5 by @dependabot[bot] in [#8](https://github.com/holochain/holochain-hdi/pull/8)
- *(deps)* Bump rand from 0.9.5 to 0.10.2 by @dependabot[bot] in [#7](https://github.com/holochain/holochain-hdi/pull/7)
- *(deps)* Bump sha2 from 0.10.9 to 0.11.0 by @dependabot[bot] in [#10](https://github.com/holochain/holochain-hdi/pull/10)
- *(deps)* Bump base64 from 0.22.1 to 0.23.1 by @dependabot[bot] in [#9](https://github.com/holochain/holochain-hdi/pull/9)
- *(deps)* Bump mockall from 0.13.1 to 0.15.0 by @dependabot[bot] in [#6](https://github.com/holochain/holochain-hdi/pull/6)
- *(deps)* Bump Swatinem/rust-cache by @dependabot[bot] in [#5](https://github.com/holochain/holochain-hdi/pull/5)
- *(deps)* Bump rust-toolchain from 1.96.1 to 1.98.1 by @dependabot[bot] in [#4](https://github.com/holochain/holochain-hdi/pull/4)

### First-time Contributors

- @dependabot[bot] made their first contribution in [#8](https://github.com/holochain/holochain-hdi/pull/8)

## \[[0.9.0-dev.4](https://github.com/holochain/holochain-hdi/commits/v0.9.0-dev.4)\] - 2026-09-08

### Features

- *(hc)* Accept modifier overrides in hc dna hash (#5969) by @veeso
  - Feat(hc): accept modifier overrides in hc dna hash
  - Hc dna hash previously always hashed the manifest's baked-in modifiers, so there was no way to compute the hash a DNA will actually have once install-time overrides are applied.
  - Add --network-seed/-s and --role-settings, matching the flags of hc sandbox generate. The role settings file holds a modifiers block for a single DNA and its network_seed takes precedence over --network-seed, mirroring installation. The overrides are applied via DnaBundle::into_dna_file.
  - Also adds an end-to-end sandbox test asserting the CLI hashes match the hashes of installed cells.
- *(hc)* Export the TypeScript bindings with an hc subcommand (#5959) by @veeso
  - Feat(hc): export the TypeScript bindings with an hc subcommand
  - Replace the ts_rs-gated export-ts-bindings binary with a built-in hc export-ts-bindings subcommand, so every hc build (including Holonix's stock packages.hc) can write the conductor API TypeScript bindings for holochain-client-js without enabling a feature or naming a second binary. The output directory is a flag (--out-dir, default ./bindings); the tree is staged and only replaces the directory once the export succeeds, and the command refuses to replace the working directory or an ancestor.
  - Hc now depends on holochain_conductor_api with ts_rs always on, which through feature unification compiles the type crates with ts_rs in every workspace build. To keep that side-effect free, drop the export flag from every #[ts(export, export_to = ...)] (export_to alone still places the declaration; the tree is driven by the export_ts_bindings chain), remove the two manual export tests and the .cargo/config.toml env block that only served them.
  - Unstable-countersigning stays an hc feature: a workspace build would unify it into holochain's holochain_conductor_api while holochain's own feature is off, and its exhaustive match on AppRequest would not compile. make ts-bindings enables it; the ts-bindings-test target now runs the hc tests that check the written tree.
- Export the conductor API types to TypeScript with ts-rs (#5934) by @veeso
  - Feat: export the conductor API types to TypeScript with ts-rs
  - Add an opt-in ts_rs cargo feature across the API type crates that generates TypeScript declarations for the conductor admin and app interfaces, with hash types exported as byte arrays, timestamps as plain numbers, and kitsune2 network types replaced by holochain-owned mirrors. The export directory is chosen via TS_RS_EXPORT_DIR (scripts/export-ts-bindings.sh / make ts-bindings).
- Upgrade Kitsune2 to 0.5.0 (#5913) by @ThetaSinner
- *(hdi)* TypedAction narrowing and agent-key ergonomics (#5910) by @ThetaSinner
  - Feat(hdi): let TypedAction convert between Create/Update and EntryCreationData
  - Scaffolded validation code needs to share one function between the create and update path for an entry type. Add IntoEntryCreationData (mirroring IntoActionData) so TypedAction<CreateData>/<UpdateData> convert into TypedAction<EntryCreationData> via .into(), plus a try_from_action helper that narrows a fetched Action straight into an ExternResult for use in a validate callback's ?-chain.
  - Also add Deref<Target = D> for TypedAction<D>, following the existing HoloHashed/RenderedOp precedent of dereferencing to the payload field, and simplify OpLink's getters to drop the now-redundant .data.
  - Feat(hdi): narrow Action to single action-data variants, restore agent key fields
  - Add TryFrom<Action>/try_from_action for CreateData, UpdateData, DeleteData, CreateLinkData, and DeleteLinkData, mirroring the existing EntryCreationData narrowing, so a freshly-fetched Action (e.g. from must_get_valid_record) can be narrowed to a specific TypedAction<D> without hand-matching ActionData and reconstructing it.
  - Also restore the agent/new_key/original_key fields that CreateAgent/ UpdateAgent variants of OpEntry, OpActivity, and OpRecord carried before the TypedAction refactor. They'd been replaced by Option- returning .agent()/.new_key()/.original_key() methods that force an unreachable-error branch even when the variant is already known from a match arm; those methods are now dropped in favour of the direct, non-optional fields, computed once at construction time.
  - Docs(hdi): note that try_from_action failures are faults, not invalid data
  - A narrowing failure here means a sys-validation-guaranteed invariant didn't hold, not that the author's data is bad. Propagating it as an error (rather than returning ValidateCallbackResult::Invalid) avoids incorrectly warranting the author for what is actually a fault in how the op reached the callback.
  - Also fixes an unrelated broken intra-doc link on EntryCreationData::try_from_action (bare `ExternResult` wasn't in scope; needed the crate::prelude:: path).
  - Docs: split run-on doc-comment summaries; note the rustdoc convention
  - Rustdoc reuses everything up to the first blank `///` line as the item's summary in module/search listings, so a multi-sentence opening paragraph with no break renders as an oversized summary. Split the run-on comments introduced on this branch into a short first-line summary plus a separated detail paragraph, and record the convention in CLAUDE.md so new doc comments follow it.
  - Refactor(hdi): use let-else in single-variant Action narrowing
  - Simplifies the Create/Update/Delete/CreateLink/DeleteLink TryFrom<Action> impls to a let-else instead of a match with a single success arm, per review feedback on #5910.
- Restore validation receipts behavior for published ops by @jost-s
  - Chore!: bump kitsune2 to 0.5.0-dev.4
  - Feat!: request validation receipts for published ops only
- Migrate to v2 action/record data model (#5730) (#5822) by @ThetaSinner
  - Docs: design for v2 data-model migration (phase 0)
  - Refactor(integrity): scaffold dht_v2 record/op submodules
  - Feat(integrity): add v2 Record type
  - Feat(integrity): add ActionData-based v2 Op with validating constructors
  - Feat(integrity): add v2 Op header/data accessors
  - Test(integrity): serde + hashing tests for v2 Record and Op
  - Refactor(hdi): scaffold flat_op_v2 staging module
  - Feat(hdi): add v2 OpRecord over dht_v2::Action
  - Feat(hdi): add v2 OpEntry/OpUpdate/OpDelete over dht_v2::Action
  - Feat(hdi): add v2 OpActivity over dht_v2::Action
  - Feat(hdi): add v2 FlatOp enum over dht_v2 op types
  - Style(hdi): cargo fmt flat_op_v2 submodules
  - Refactor(hdi): expose flatten helpers and scaffold op_v2 trait
  - Feat(hdi): flatten v2 Op into v2 FlatOp
  - Style(hdi): cargo fmt op_v2 and widened helper signature
  - Test(hdi): cover v2 flattened conversion
  - Style(hdi): enforce missing_docs on v2 staging modules
  - Docs: design for v2 migration phase 1 (cascade reads + wire cutover)
  - Feat(state): DhtStore::retrieve_action
  - Feat(state): DhtStore::retrieve_entry
  - Feat(state): DhtStore::retrieve_record
  - Feat(state): DhtStore::get_live_record
  - Feat(holochain_data): add get_live_entry_creates query
  - Feat(state): DhtStore::get_live_entry
  - Feat(holochain_data): add delete/update-actions-for-record queries
  - Feat(state): DhtStore::get_record_details
  - Feat(holochain_data): add entry-creates and by-entry delete/update queries
  - Feat(state): DhtStore::get_entry_details
  - Feat(holochain_data): add get_live_link_actions query
  - Feat(state): DhtStore::get_links
  - Feat(holochain_data): add link-create and delete-link-actions queries
  - Feat(state): DhtStore::get_link_details
  - Docs(v2-migration): design agent-activity 1a reads
  - Pin the design for the highest-risk 1a reads: split agent activity into 1a-viii (get_agent_activity, executed first) and 1a-ix (must_get_agent_activity dht-only core), both store-only. Fix the SQL-vs-Rust boundary (scans in holochain_data; pure assembly moved from holochain_cascade into holochain_state), the rich LEFT JOIN Entry scan for Full mode, single Full/Hashes DhtStore method, separate warrant fetch, and legacy return boundary.
  - Docs(v2-migration): agent-activity reads scan ChainOp only
  - Settle the pending-ops question: v2 get_agent_activity scans the integrated ChainOp table only (validated activity), dropping the legacy behavior where pending LimboChainOp ops raised highest_observed. Pin the holochain_state-local options struct (GetActivityOptions lives in holochain_p2p, off the holochain_state dep graph) and reuse the existing get_warrants_by_warrantee primitive.
  - Feat(holochain_data): add get_agent_activity scan
  - Feat(state): DhtStore::get_agent_activity
  - Test(state): get_agent_activity rejected, fork, full, warrants
  - Feat(holochain_data): add must_get_agent_activity scans
  - Feat(state): DhtStore::must_get_agent_activity
  - Test(state): must_get_agent_activity response variants
  - Docs(v2-migration): reshape Phase 1 tail around data-serving model
  - Data serving is about records, not ops: serve actions/entries with a record-level validation status + warrants, with the invariant that a rejected record always carries a proving warrant (checked up front by the receiver). The receiver expands valid records to ops (produce_ops_from_record) filtered by request type + storage arc and re-validates locally; rejected+warranted records go to limbo.
  - Because the serving wire type is shared by authority, holochain_p2p, and requester, reshaping it is atomic across the three crates, so the old authority (1b) and requester (1c) slices merge into one phased-commit slice 1b. The old gossip wire-break becomes the new final phase 1c.
  - Fix(state): integrate_ready_ops populates action indexes
  - Test(state): integration indexes delete-link
  - Feat(holochain_data): authority link reads (locally_validated)
  - Feat(state): DhtStore authority link reads
  - Fix(holochain_data): upgrade cached ChainOp to locally_validated on integration
  - Feat(holochain_data): authority record reads (locally_validated)
  - Feat(state): DhtStore authority record reads
  - Feat(holochain_data): authority entry reads (locally_validated)
  - Feat(state): DhtStore authority entry reads
  - Docs(v2-migration): design the 1b-vi data-serving wire reshape
  - The authority-serving DhtStore reads (locally_validated-guarded) are done; what remains is the wire cutover. Record the finding that the wire reshape and the requester's expand-to-ops are the same change (WireOps::render becomes produce_ops_from_record), so they are one slice; the reshaped wire shape (Judged<SignedActionHashed> + warrants, dropping the op-shaped forms); and the decision to land it as one monolithic, tightly-coupled commit across holochain_types + holochain_cascade + holochain_p2p, executed inline.
  - Feat(cascade): serve records (not ops) on the data-serving wire
  - Reshape the get authorities' data-serving wire from op-shaped forms to records. WireRecordOps/WireEntryOps/WireLinkOps now carry Judged<SignedAction> (with their record-level validation status) plus a warrants field, instead of the op-shaped WireDelete/WireCreateLink/etc. A Rejected record is always paired with a warrant proving it; the requester checks that invariant up front so a malicious peer cannot force pointless validation work, before caching Valid ops + warrants.
  - The authority handlers are rewritten onto the locally-validated DhtStore reads (get_authority_*) + retrieve_entry, with the production Cell caller passing space.dht_store.as_read(); a public get_warrants_by_warrantee is added to DhtStoreRead for warrant pairing. WireOps::render now rebuilds the request-relevant op per served action rather than reassembling op-shaped forms.
  - Removes the legacy authority Query structs and the legacy-DbKindDht cascade integration-test layer (PassThroughNetwork + op-shaped test-data builders), which could not be bridged to a holochain_data DhtStore; focused new-path unit tests cover the rejected-requires-warrant invariant. The requester's local read path keeps CascadeTxnWrapper for now (a separate cutover). See docs/design/v2_model_migration.md for the deviations and follow-ups.
  - Docs(v2-migration): design the 1c op + action hash cutover
  - Expand the Phase 1 1c section into the full op + action hash cutover: action and op hashes become content-derived v2 (no weight), op construction moves to the v2 dht_v2 op types, the gossip wire carries v2 ops, and the network-path legacy<->v2 conversions + hash-preservation hack are deleted.
  - Records it as an identity flip (coordinated, green only at the end) rather than additive reads, with two additive foundation slices (1c-i v2 op-hash, 1c-ii v2 produce-ops) preceding the coordinated cutover (1c-iii). Notes the source-chain action-hash seam as the corner to trace when planning 1c-iii.
  - Feat(types): content-derived v2 op hash (ChainOp::to_hash)
  - Feat(types): v2 op-hash/basis/op-type primitives + AnyLinkableHash basis
  - Feat(types): v2 produce_ops_from_record -> Vec<HashedChainOp>
  - Docs(v2-migration): resolve the 1c-iii boundary
  - Record the 1c-iii scope decision: "data-layer v2, authoring keeps legacy types". Action and op hashes become content-derived v2 by flipping the canonical hash (HashableContent for Action + the op ChainOpUniqueForm) to the weightless v2 projection, so everything on disk and wire is v2 and prev_action chains in v2, while the ActionBuilder/scratch/ribosome keep legacy Action types (full v2-native authoring stays Phase 3).
  - Notes the shape: a single coordinated identity flip (green only at the end, executed inline like 1b-vi), with test fallout as the bulk of the work; dropping weight from the hash is collision-safe and the v2 projection is total via from_legacy_signed_action.
  - Feat: content-derived v2 action/op hashes + v2 gossip wire
  - Flip the canonical ActionHash and DhtOpHash to the v2 projection of the action (ActionHeader + ActionData, no weight), so data on disk and on the gossip wire carries v2 hashes and prev_action chains in v2. Authoring still builds legacy Action types; only the hashed identity is flipped (full v2-native authoring stays Phase 3).
  - Add holochain_integrity_types::dht_v2::from_legacy_action, the total   legacy->v2 action projection; from_legacy_signed_action reuses it. - HashableContent for the legacy Action, the per-ref action impls, and   EntryCreationAction now hash the v2 projection (drop the dead ActionRef). - The legacy op ChainOpUniqueForm hashes the v2 projection (via the v2   ChainOpUniqueForm::op_hash); route ChainOp::hashable_content through the   unique form so both hash paths agree byte-for-byte with ChainOp::to_hash. - op_store: encode/decode the v2 DhtOp on the gossip wire natively, building   it straight from the stored rows (delete the legacy reconstruction); op ids   use the native v2 rehash. Add dht_v2::DhtOp::{to_hash,dht_basis} and   dht_v2::to_legacy_dht_op. - Ripple handle_publish / the p2p event signatures to Vec<dht_v2::DhtOp>;   Space::handle_publish reconstructs the legacy op for the still-legacy DHT   table + record_incoming_ops during the migration.
  - Docs(v2-migration): mark 1c-iii / Phase 1 complete
  - 1c-iii (the content-derived v2 action+op hash flip + v2 gossip wire) has landed, completing the 1c sub-slice and Phase 1. Records the one remaining transitional bridge (receive-side legacy reconstruction for the still-legacy ingest), which Phase 2 removes.
  - Feat(state): retrieve_* scratch overlay reads
  - Feat(state): get_live_* scratch overlay reads
  - Feat(state): get_*_details scratch overlay reads
  - Feat(state): get_links* scratch overlay reads
  - Refactor(state): address CR-1 scratch-overlay review
  - Collapse `scratch_deletes_targeting` into `scratch_delete_targets().contains()`   (one scratch-scan helper, not two). - Use the module-level `RecordValidity` / `ValidationStatus` imports in the   details overlays, matching their store-only siblings. - Note that `scratch_creates_for_entry` includes tombstoned creates (vs   `scratch_live_entry_creates`), since `EntryDetails.actions` lists all creates.
  - Feat(state): get_agent_activity scratch overlay
  - Feat(state): must_get_agent_activity scratch overlay
  - Refactor(state): address CR-1-aa agent-activity review
  - Convert the action once per item in `get_agent_activity_with_scratch` (match   on validity first, dropping the redundant `to_legacy_signed_action` calls). - Document that valid records carry no entry (cascade `cached_entry: None`   convention) while rejected records keep theirs. - `agent_activity_from_scratch` takes `Option<u32>` for the chain-top bound   instead of a `u32::MAX` sentinel.
  - Feat(cascade): wire live record/entry reads to DhtStore
  - Feat(cascade): wire details reads to DhtStore
  - `get_entry_details` and `get_record_details` now take the `dht_store`-present path: optional network fetch (honouring `GetStrategy`/`am_i_authoring`/`am_i_an_authority` gating) followed by a local read via `DhtStoreRead::get_entry_details_with_scratch` / `get_record_details_with_scratch`.  The legacy `get_latest_with_query` path is kept as the `dht_store`-absent fallback until CR-3.
  - Two new tests in `dht_store_scratch_overlay_tests` cover: - `get_record_details_reflects_scratch_delete` — scratch Delete on an   integrated record appears in `RecordDetails.deletes`. - `get_entry_details_reflects_scratch_delete` — scratch Delete flips   `entry_dht_status` to Dead in `EntryDetails`.
  - Feat(cascade): wire link reads to DhtStore
  - Feat(cascade): wire agent-activity reads to DhtStore
  - Fix(state): hide private entries from non-authors in record reads
  - The requester record reads attached an entry resolved purely by hash, so a private entry could leak to a non-author whenever that caller happened to hold a same-hash private entry of their own (the entry hash is shared by content, but the privacy is per author). The by-hash `get_entry` lookup alone does not provide the legacy read's entry-visibility hiding.
  - `retrieve_record` / `retrieve_record_with_scratch` now attach a private entry only when the caller is the action's author; otherwise the entry is `Hidden` (the action itself stays visible). Regression test `retrieve_record_hides_private_entry_from_non_author`.
  - Fix(cascade-read): resolve full-suite regressions from the DhtStore cutover
  - The CR-2 cascade read cutover (local reads now via DhtStore *_with_scratch) surfaced 16 failures across workflows, host fns and sweettests. Two were real read-path bugs in holochain_state; the rest were tests seeding the legacy DBs the cascade no longer reads.
  - Production read fixes (dht_store/reads.rs): - get_record_details_with_scratch gated on an integrated StoreRecord op before   consulting the scratch, so a scratch-only record (an author's just-created   entry mid-call) returned None. This broke in-call update->create resolution   and, via stalled validation dependency resolution, the consistency   sweettests and multi_create_link_validation. Resolve the record first;   scratch-only records are Valid from the author's view. - get_links / get_links_with_scratch sort by timestamp, matching the legacy   GetLinksQuery creation-order contract.
  - Test seeding migrated to the DhtStore (the cascade's new local source): - get_zomes_to_invoke, main_workflow, run_validation_callback,   must_get_valid_record_short_circuit, adds_init_marker now seed via   record_incoming_ops / cache_chain_ops (+reject_chain_ops) / genesis-into-store   instead of insert_op_dht / insert_op_cache. reject_chain_ops only transitions   network-cached ops, so rejected-record cases use cache_chain_ops (integrated),   not record_incoming_ops (limbo). - must_get_entry rewritten to reject a cached (network-fetched) record rather   than an authored op (reject_chain_ops is a no-op on locally-validated ops by   design) -- the realistic rejected-record scenario.
  - Add a regression test that the op hash is weight-independent: the v2 action projection drops weight, so the op hash must survive the v2 round-trip.
  - Refactor(cascade): remove the legacy dht_store-None read path (CR-3)
  - Now that CR-1/CR-2 made the DhtStore the live local-read source, delete the legacy fallback that is no longer reachable in production:
  - Make `dht_store` mandatory on `CascadeImpl` (drop the `Option`); `empty` now   takes a `DhtStore`. Remove the read-only `authored`/`dht` `DbRead` fields and   the `with_authored`/`with_dht` builders. - Delete the `dht_store`-`None` fallback arms in every read method, plus the   legacy read helpers they used (`cascading`, `find_map`,   `get_local_first_with_query`, `get_latest_with_query`) and the now-dead   cascade duration metric. - Delete the unused `authority::get_agent_activity_query` module. - Update cascade constructors at their call sites (sys_validation   `local_cascade`/`network_and_cache_cascade`, app_validation `full_cascade`,   countersigning) to the `empty(dht_store)` shape. `local_cascade` previously   omitted the DhtStore and silently used the legacy path.
  - The `cache` field and the network-fetch cache write (`merge_ops_into_cache`) are kept; `CascadeTxnWrapper` / `DbScratch` / `Query` in holochain_state remain (still used by source_chain authoring) -- their removal is a later phase.
  - Fix two sys-validation unit tests to seed a held dependency as a cached op (`save_chain_op_as_cached`) rather than via `record_incoming_ops`: the latter queues the dependency itself for sys validation, so the workflow would try to validate it and fetch its prev_action from the network.
  - Refactor(publish): read the publish queue from the DhtStore (Phase 2a)
  - Move the publish workflow off the legacy authored database onto the v2 DhtStore. `get_ops_to_publish` / `num_still_needing_publish` are now SQL queries over ChainOp/Action/ChainOpPublish (and WarrantOp/Warrant/ WarrantPublish) in holochain_data, surfaced as DhtStoreRead methods.
  - The private-entry leak guard is preserved verbatim: StoreEntry ops whose action carries a private entry are never returned for publishing.
  - Warrants live in a separate publish table in the v2 schema, so record_published_op_hashes now records warrant publish times in WarrantPublish (warrants still publish once); the legacy single DhtOp.last_publish_time covered both.
  - The publish workflow and its consumer no longer take a DbKindAuthored handle. The legacy publish_query module is removed; its coverage moves to DhtStore-backed tests beside the workflow.
  - Refactor(genesis): read genesis status from the DhtStore
- Parallel-write DHT slice (#5771) by @ThetaSinner
  - Feat(holochain_data): add withhold_publish to ChainOpPublish
  - Nullable INTEGER column. Required by the parallel-write DHT slice to mirror the legacy countersigning withhold flag without changing the publish-row presence model.
  - Feat(holochain_data): add LimboChainOp status setters
  - Set_sys_validation_status, set_app_validation_status, set_abandoned_at exposed on DbWrite<Dht> and TxWrite<Dht>. Used by the DHT mirror in holochain_state to record validation outcomes.
  - Feat(holochain_data): add LimboChainOp.set_require_receipt
  - Feat(holochain_data): add LimboWarrant status setters
  - Feat(holochain_data): add limbo→terminal promotion ops
  - Implements `promote_limbo_chain_op` and `promote_limbo_warrant` on `DbWrite<Dht>` and `TxWrite<Dht>`.  Each op atomically SELECTs the limbo row, INSERTs into the terminal table (`ChainOp` / `Warrant`), and DELETEs the limbo row in a single transaction.  Returns `bool` indicating whether the limbo row existed.
- Parallel writes from authored DB to new holochain_data DHT DB (#5729) (#5759) by @ThetaSinner
  - Feat(integrity): add CloseChain/OpenChain variants to dht_v2 ActionData
  - Extends ActionType to 1..=10 and adds CloseChainData / OpenChainData mirroring the legacy struct field shapes (new_target / prev_target / close_hash) so chain-migration actions can be represented in the new DHT schema.
  - Feat(holochain_data): DHT schema and API for parallel-write authored slice
  - Add ScheduledFunction table (per-author within a per-DNA DB; PK   ordered (author, zome_name, scheduled_fn) for clustered range scans). - Apply ON CONFLICT IGNORE to every content-addressed primary key so   duplicate inserts are idempotent (matches the legacy schema's   semantics for Action / Entry / DhtOp). - Surface insert/delete operations on DbWrite<Dht> and TxWrite<Dht> for   ScheduledFunction, the Link/DeletedLink/UpdatedRecord/DeletedRecord   index tables, and ChainOpPublish.set_receipts_complete. - Index inserts take Insert* parameter structs matching the   InsertChainOp shape; writes return rows-affected. - New holochain_data tests cover the new operations.
  - Feat(holochain_state): add DhtStore and mirror authored writes to it
  - Introduce a per-DNA DhtStore that wraps the new holochain_data DHT DB and exposes domain-meaningful operations (reschedule_expired_persisted, upsert/unschedule, mark_chain_op_receipts_complete, purge_all). Plumb the store through SourceChain so genesis and flush mirror their writes into the new DB after the legacy authored DB transaction commits.
  - DhtStoreError unifies the underlying sqlx and schedule error types. - compute_schedule_params and serialize_maybe_schedule helpers replicate   the legacy schedule_fn formula on the new DB side. - Strict-on-failure: a mirror error propagates as a real Err to the   caller rather than being swallowed. - Genesis and flush both compute serialized_size by encoding the full   DhtOp at write time (no sentinel zeros).
  - Feat(holochain): wire DhtStore through Space, Cell, conductor and workflows
  - Space::new opens the per-DNA holochain_data DHT DB and constructs a   DhtStore alongside the legacy DBs. - DhtStore is threaded through SourceChain construction, the genesis   and call-zome workflows, the app-validation workspace, and every   test fixture / sweettest helper that builds those. - Cell::dispatch_scheduled_fns and Cell::handle_validation_receipts_received   call the new store directly (delete_live_ephemeral_scheduled_functions,   reschedule_expired_persisted, upsert_scheduled_function,   unschedule_function, mark_chain_op_receipts_complete) rather than   reaching for the underlying database. - Conductor::delete_cell_databases drops the mirrored DB file when the   last cell for a DNA is uninstalled, falling back to DhtStore::purge_all   if the file is still locked. The legacy purge dance is mirrored.
  - # Conflicts: #	crates/holochain/src/conductor/space.rs
  - Docs: CLAUDE.md guide and state_model.md updates for the parallel-write slice
  - Add CLAUDE.md describing build/test commands, architecture, project   conventions, the workflow set, the offline-friendly principle, and   the in-progress migration to holochain_data. - Document the new ScheduledFunction table in state_model.md and extend   the action_type discriminant range to cover CloseChain (9) and   OpenChain (10).
- New DHT database schema and skeleton API (#5743) by @ThetaSinner
  - Feat(integrity-types): scaffold dht_v2 with RecordValidity and ActionType
  - Feat(integrity-types): add dht_v2 ActionHeader and per-variant data structs
  - Feat(integrity-types): add dht_v2 ActionData and Action with HashableContent impl
  - Feat(zome-types): add dht_v2 module with SignedAction/SignedActionHashed and ChainOpType i64 mapping
  - Fix(zome-types): correct ChainOpType 6/7 mapping to match state_model ordering
  - Test(zome-types): pin ChainOpType i64 mapping with forward-direction assertions
  - Feat(types): add dht_v2 OpEntry, ChainOp, WarrantOp and DhtOp
  - Feat(types): add dht_v2 HashedChainOp
  - Feat(holochain-data): wire DbKind::Dht and DHT_MIGRATOR
  - # Conflicts: #	crates/holochain_data/src/kind.rs #	crates/holochain_data/src/lib.rs
  - Feat(holochain-data): add initial DHT schema migration
  - Feat(holochain-data): add DHT row models
  - Feat(holochain-data): add Action insert/read primitives with round-trip tests
  - Feat(holochain-data): add Entry and PrivateEntry insert/read with isolation and tx tests
  - Feat(holochain-data): add CapGrant and CapClaim operations with FK test
  - Feat(holochain-data): add ChainLock acquire/read/release/prune operations
  - Feat(holochain-data): add LimboChainOp insert/read/delete with state-based queries
  - Feat(holochain-data): add LimboWarrant insert/read/delete with state-based queries
  - Feat(holochain-data): add ChainOp insert/read with basis and action queries
  - Feat(holochain-data): add Warrant insert/read with warrantee lookup
  - Feat(holochain-data): add ChainOpPublish, WarrantPublish, and ValidationReceipt operations
  - Feat(holochain-data): add Link/DeletedLink/UpdatedRecord/DeletedRecord operations with CASCADE test
  - Test(holochain-data): cover every ActionData variant through round-trip
  - Docs(holochain-data): add changelog entry for DHT schema and skeleton API
  - Style: cargo fmt and elide needless lifetimes
  - Chore: review changes
  - Chore: review changes
  - Chore: remove redundant 'TxWrite-only' module comments
  - Docs(holochain-data): replace private intra-doc links with code formatting
  - The static-doc check uses RUSTDOCFLAGS=-Dwarnings, and rustdoc was warning that public docs link to private items (`inner`, `db_operations`, `tx_operations`, and `super::super::inner::chain_lock::acquire_chain_lock`). Demote those references to plain code spans.
  - Chore(holochain-data): drop dead DEFAULT 0, document FK convention, close test gaps
  - Remove DEFAULT 0 from ChainOp.serialized_size (the insert always binds   it, so the default was unreachable). Mirror the change in   state_model.md so the design and schema agree. - Add a foreign-key delete-behaviour note to the migration header   explaining why the index tables cascade and the rest do not. - Add actions_by_author_excludes_other_authors covering the negative   case for the author filter, and cap_grants_ordered_by_action_seq   asserting the ORDER BY Action.seq tiebreaker for both grant queries.
  - Test: Improve `actions_by_author` and `actions_by_author_excludes_other_authors` so that actions are compared directly rather than just some fields
- Wasmer upgrade (#5717) by @ThetaSinner
  - Feat: wasmer upgrade
  - # Conflicts: #	Cargo.lock #	crates/client/Cargo.toml #	crates/hc_bundle/Cargo.toml #	crates/holochain/Cargo.toml #	crates/holochain_cascade/Cargo.toml #	crates/holochain_conductor_api/Cargo.toml #	crates/holochain_data/Cargo.toml #	crates/holochain_keystore/Cargo.toml #	crates/holochain_p2p/Cargo.toml #	crates/holochain_state/Cargo.toml #	crates/holochain_types/Cargo.toml #	crates/holochain_websocket/Cargo.toml #	crates/test_utils/wasm_common/Cargo.toml
  - # Conflicts: #	crates/holochain_data/Cargo.toml
  - Chore: progress with dependency upgrades
  - Chore: Integrate the wasmer upgrade
  - Chore: lint fixes
  - Chore: lint fixes
  - Chore: review changes
  - Chore: update Rust version for nix
  - Chore: address PR review comments
  - Chore: update weekly Rust version
  - Chore: fix build
  - Chore: review changes
  - Chore: improve changelog
- \[**BREAKING**\] Split auth material into bootstrap and relay auth material (#5697) by @jost-s
  - Feat!: split auth material into bootstrap and relay auth material
  - Refactor: redact auth material in terminal output
  - Docs: mention base64 format
  - 
- Upgrade to Kitsune2 0.4.0-dev.4 (#5676) by @ThetaSinner
- Upgrade k2 to 0.4.0-dev.3 (#5673) by @ThetaSinner
- 4761 specify bootstrap and sbd urls in the app manifest (#5524) by @veeso
- Remove unused generic `A` from `Record` (#5483) by @ThetaSinner
  - Feat: Remove unused generic `A` from `Record`
  - Docs: Update changelog
- Upgrade to kitsune2 release 0.3.0-dev.4 by @jost-s
  - Fix: work around peers being blocked after preflight in tests
  - Test: Fix report test
  - Fix: clean up warrant test & await updated storage arc in peer store
  - Refactor: add agents synchronously to peer store in preflight validation
  - Docs: fix app disabling comments in warrant_is_gossiped test
  - 
- Add hash trait implementation for X25519PubKey (#5397) by @grandima
- Add query are_all_blocked to holochain_state (#5281) by @jost-s
  - Feat: add query are_all_blocked
  - Refactor: upgrade kitsune2 to 0.3.0
  - Docs: mark block target id as doc link in query_are_all_blocked
  - Refactor: improve vector initialization in query_are_all_blocked
  - Docs: fix changelog punctuation
  - Docs: fix comment in test query_are_all_blocked_false_when_outside_of_interval
- \[**BREAKING**\] ChainFilters Until Timestamp (#5015) by @ddd-mtl
  - Modified ChainFilter to implement Until Timestamp feature (https://github.com/holochain/holochain/issues/4754)
  - 
- \[**BREAKING**\] Removed Hash host function (#5129) by @veeso
  - Feat(holochain)!: Removed `hash` from Host functions
- *(holochain_integrity_types)* Implemented `action_hash` for `Op`. (#5118)
- Update sodoken and Lair (#4858) by @ThetaSinner
  - Feat: Update sodoken and Lair
  - Docs: Update changelog
- Integrate k2 (#4791) by @ThetaSinner
  - Feat: Add local agent (#4725)
  - Feat: Add local agent
  - Chore: Update changelog
  - Chore: Build on integration PRs
  - Chore: Rename `client` to `keystore_client` in the HolochainP2pLocalAgent
  - Remove legacy kitsune (#4728)
  - Remove legacy kitsune  * remove origin_time and toml-fix  * clippy
  - Holochain implementation of op store (#4720)
  - Feat: Implement OpStore for Holochain  * feat: Implement slice hash logic for the Holochain op store  * feat: Update host integration to "publish" ops rather than storing them directly  * chore: Clippy fix  * chore: Clippy fix  * chore: Clippy unstable fixes  * chore: Review changes  * chore: Review changes  ---------
- Integration only happens in the integrate workflow (#4605) by @cdunster
  - Feat: add integration of StoreRecord ops to integrate workflow * feat: add integration of StoreEntry ops to integrate workflow * feat: remove integrate from app validation and authored ops workflows * docs: update doc-comment about CreateLink Action * feat: add integration of any RegisterAddLink ops to integrate workflow We now integrate all valid create link actions instead of only ones that link to a StoreEntry.
  - Fix: wasn't integrating genesis actions when had others to integrate When nothing had been integrated yet then we started integration at the value of how many ops were `ready_to_integrate` so if we had other ops that were ready then the range started at them instead of at genesis (index 0).
  - Test: print every inconsistent agent when await_consistency fails * refactor: rename SQL query to set add link ops to integrated * refactor: rename SQL query to set delete link ops to integrated * refactor: rename test function to register_agent_activity_create_link * test: add test for RegisterAgentActivity op for Dna action * test: add test for RegisterAgentActivity op for DeleteLink action * test: add tests for range returned by get_activity_to_integrate * test: add test for RegisterAgentActivity op for AgentValidationPkg * test: add test for RegisterAgentActivity op for InitZomesComplete * test: add test for RegisterAgentActivity op for CloseChain action * test: add test for RegisterAgentActivity op for OpenChain action * test: add test for RegisterAgentActivity op for Create action * test: add test for RegisterAgentActivity op for Update action * test: add test for RegisterAgentActivity op for Delete action * test: remove some unnecessary steps from RegisterAgentActivity tests * docs: update comment about ready to integrate inclusive range * test: add check that op is valid but not integrated * refactor: rename activity -> agent_activity_op * refactor: rename vector agent_activity -> agent_activity_ops * feat: add flow to integrate warrant ops as part of integration workflow
- Generate conductor config json schema (#4569) by @c12i
  - Implement schemars for ConductorConfig and associated types  * Generate conductor config schema file on holochain build  * Revert tust-toolchain.toml  * Move url2schema to utils crate  * Apply code review suggestions  * Revert schema generation in holochain build script and move functionality to holochain cli  * Remove conductor api as dep to holochain  * Update changelog  * Update cargo lock
- Don't use wasmer module cache with wasmer_wamr feature (#4441) by @mattyg
  - Feat: don't use wasmer module cache with wasmer_wamr feature  * feat: bypassing module cache in wamr mode working  * build: bump holochain_wasmer_* crates  * chore: taplo  * fix: RealRibosome construction with optional cache  * build: bump proptest & proptest-derive  * wip: attempt to fix macos-latest wasmer_wamr building  * build: install ninja for macos wasmer_wamr ci runs  * fix: disable test that wasm cache works in wasmer_wamr mode  * chore: unused import  * test: increase maximum response time to concurrent zome calls in wasmer_wamr mode, where zome calls are expected to take longer due to wasm interpreting  * chore: changelogs  * fix: test assertion message  * chore: clearer doc comment  * build: fix holochain_wasmer_host feature
- *(hdk)* Add call get_agent_key_lineage (#4215) by @jost-s
  - Fix app validation re-trigger delay  * add key lineage host fn  * uniformly use dpkiapi type  * add zome tests for key lineage  * add tests for calling key lineage from init  * update changelog  * indicate where tests can be found  * styel: remove redundant closure  * panic when dpki is called from wrong context  * fix  * Update crates/hdk/src/agent.rs
- *(Cargo)* Add clippy lints to workspace and enable for holochain (#3318) by @steveej
  - Feat(Cargo): add clippy lints to workspace and enable for holochain  * feat(nix/holochain/clippy): read args from workspace and remove nix config file  by reusing the workspace lints we avoid the CI requirement of checking for the inheritance of lints in each crate.  * Do legwork to add lints section to crates...  ---------
- *(CHANGELOGs)* Facilitate bump to next minor pre-release (#1823) by @steveej
- Mark all crates for minor version bump (#1757) by @steveej
- \[**BREAKING**\] *(cap-grant)* Add wildcard for zomes and fns (#1732) by @zippy
  - Fixed signed_zome_call test  * updates to convert GrantedFunctions to enum with All and Listed  * added change log for Cap grant update  * add 'arbitrary' attribute to GrantedFunctions  * fix release workflow  * remove unused import
- Set all frontmatters to facilitate the 0.1.0-beta-rc.0 bump (#1717) by @steveej
  - The changes were autogenerated using the following command:  ``` $ nix-shell --pure --argstr flavor release --run 'release-automation \   --workspace-path=$PWD --log-level=debug --match-filter=".*" changelog \   set-frontmatter <(cat <<EOF default_semver_increment_mode: !pre_minor beta-rc EOF ) ' ```
- \[**BREAKING**\] *(app-api)* Require zome calls to be signed (#1510) by @thedavidmeister
  - Wip signed zome calls  * wip on signed calls  * wip on signed calls  * signed zome calls is compiling  * fix signed zome call tests  * fix tests  * wip on nonces  * wip on nonce in conductor  * nonces compiling  * compiling nonce expiries  * fix compile issues  * dont witness fresh nonce  * zome call witness nonce tests  * fix tets  * better error message in app interface  * fix tests  * lint  * fix tests  * debug  * fix test  * hash signed call data  * fix tests  * add tests for zome call auth  * changelog  * reorder checks  * remove package lock  * fix duplicate imports & version conflicts  * fix conductor tests  * fix more conductor handle methods  * fix compilation  * move unsigned zome call into zome types  * lint  * add blake2b-256 export to holo_hash  * debug ci  * add feature "encoding" to holo_hash import in holochain_zome_types  * extend key authorization to signed zome call test  * impl from, debug, eq for Nonce256Bits  * add failing test of signed zome call w/o cap secret  * update keystore dep in conductor_api  * impl visitor in secure primitives macro  * delete duplicate secure_primitive macro from zome_types  * clippy fixes: do not clone nonce  * comment out failing signed zome call test  * enable encoding feat for holo_hash in zome_types  * Revert "delete duplicate secure_primitive macro from zome_types"  This reverts commit 2fe9c8db0e6ceefce99681849f83b4ad7b6d80e7.  * fix holochain websocket integration tests  * cleanup websocket integration tests  * refactor has_initialized to check for init marker  * remove unused import  * fix error & simplify has_initialized check  * extend nonce expiry in signed zome call test  * fix clippy field reassign with default in source_chain  * replace zome init complete query by explicit field  * revert to query source for init zomes complete  * fmt  * simplify equality check against true in source_chain  * warn about multiple init zome complete actions in source_chain
- *(admin-api)* Add calls to authorize signing key & get dna def (#1641) by @jost-s
  - Adding 2 calls to the Admin API: * authorize public key to authorize zome calls * get DNA definition
- *(hdi)* Mark for minor version bump by @steveej

### Bug Fixes

- Restore Kitsune2 0.5.0-dev.6 upgrade (+ Windows QUIC pin) (#5849) by @ThetaSinner
  - Fix: restore Kitsune2 0.5.0-dev.6 upgrade with quinn/socket2 pin
- Remove uses of dependencies as features (#5663) by @ThetaSinner
- Handle op ids with invalid length gracefully (#5359) by @matthme
- Inconsistent DHT locations reported to Kitsune2 (#5167) by @ThetaSinner
  - Fix: Inconsistent DHT locations reported to Kitsune2
  - Chore: Update naming and docs
- More descriptive error messages and remove unnecessary if/else clause that caused inconsistent behavior (#5105) by @matthme
  - Fix: remove unnecessary if/else clause that caused inconsistent behavior, improved error message if conversion to ScopeLinkType or ScopedEntryDefIndex fails
  - Fix: add required dependencies argument to InlineZomeSet constructors, fix missing zome dependency values
- Add missing description to root_hash() function (#5084) by @matthme
- Compile error when `hdk_extern` is used with wrong type (#4617) by @c12i
  - Check hdk_extern annonatated function return types  * Remove migrate_agent handling  * Fix clippy warning  * Handle post_commit  * Update CHANGELOG  * Better name util  * Remove entry_defs check  * Better context with diagnostics  * Add tests  * Fix clippy warnings  * Add validate infallible test and rename genesis self check infallible test  * Improve infallible suggestion  * Remove out of scope testcase  * Move tests to `hdk` crate, replace compiletest_rs with trybuild  * Fix hdk_extern_infallible_invalid_return test  * Fix failing tests and update hdk changelog  * Improve diagnostic messages  * Replace validate_invalid_return test_wasm test with inline zome  * Replace init_invalid_return test_wasm test with inline zome  * Remove unused TestWasm variants  * Revert unwanted diffs  * Remove un-required testwasm test
- Wasmer memory access fault & caching (#3152) by @jost-s
  - Test(ribosome): add guard for zome call response time  * test(ribosome): add cache test  * test(ribosome): add test for module and instance cache  * remove unused import  * test: add concurrent zome call test of response times  * refactor(real-ribosome): remove instance caching fn  * refactor(real-ribosome): remove instance cache altogether  * refactor(real-ribosome): move cache logic to holochain-wasmer  * refactor: move all wasmer-types items to holochain-wasmer  * refactor(real-ribosome): change identifier names according to refactors  * fix: clean up contexts from memory after calls  * docs: add changelogs  * doc: update changelog  * refactor: move ios precompiled module getter to holochain-wasmer  * refactor: rename precompiled module fn  * build: remove wasmer dependencies from holochain_types  * test: fix concurrent zome call test  * docs: update breaking change in changelog  * build: update holochain wasmer crates to v0.0.92  * delete unused import  * delete unused import  * Fix symlink path mismatch  * Use the path that sqlite gives back  * Clippy fix  * fix: check if const fn exists  * refactor: simplify error handling in call_zome_fn  * lint: clippy fixy  * test: add must_get_agent_activity bug reproduction  * test: update test name  * test: update test  * fix: must_get_agent_activity wasm  * build: update both workspaces' cargo locks  * test: fix must_get_agent_activity saturation  ---------
- Remove shadowing public glob re-exports (#3092) by @jost-s
  - Fix: remove shadowing public glob re-exports  * refactor: export error types directly instead of error mod  * docs(changelog): add refactors  * docs(changelog): mark as breaking changes  * style: fmt
- Fix merge by @thedavidmeister
- Fix bad merge by @thedavidmeister
- Fix tests by @thedavidmeister
- Fix changelog by @thedavidmeister
- Fix changelog by @thedavidmeister
- Fix doc by @freesig
- Fix doc comment by @freesig
- Fix review by @thedavidmeister
- Fix check enzyme by @thedavidmeister
- Fix compiling countersigning by @thedavidmeister
- Fix ser by @neonphog
- Fix docs by @freesig
- Fix errors by @freesig
- Fix links in holochain_integrity_types by @jost-s
- Fix links in holochain_util by @jost-s
- Fix links in fixt by @jost-s
- Fix string by @freesig
- Fix trait by @freesig
- Fix integrity toml by @thedavidmeister
- Fix links and add backticks by @jost-s
- Fix call remote test by @freesig
- Fix multiple ignored tests by @freesig
- Fix update query by @freesig
- Fix clippy lints by @steveej
- Fixing manifeset documentation urls
- Fix hardest bug ever and some other perf fixes by @freesig
- Fix mutable elements by @thedavidmeister
- Fix tests by @thedavidmeister
- Prepare READMEs and Cargo.toml for releasing by @steveej
- Fixt -> Apache-2.0 by @neonphog
- Fix tests by @thedavidmeister
- Fix random tests by @thedavidmeister
- Fix tests by @thedavidmeister
- Fix fixt to work on rust latest by @freesig
- Fix broken test by @freesig
- Fix macro by @freesig
- Fix cap grant tests by @thedavidmeister
- Fix infinite compiler loops by @thedavidmeister
- Fix serialize by @freesig
- Fix all trivial clippy warnings by @timotree3
- Fix the broken async/blocking model for holo_hash by @neonphog
- Fix merge by @thedavidmeister
- Fix tests by @thedavidmeister
- Fix compilation after merge by @thedavidmeister
- Fix circular deps by @thedavidmeister

### Miscellaneous Tasks

- Update the AI_POLICY.md with shared content in [#3](https://github.com/holochain/holochain-hdi/pull/3)
- Update the CONTRIBUTING.md with shared content
- Switch to dev releases for 0.8 by @ThetaSinner
- Prepare the 0.7.0 release by @ThetaSinner
- Prepare 0.7 for RC releases (#5884) by @ThetaSinner
- Update kitsune2 dependencies to 0.4.0-dev.5 (#5690) by @ThetaSinner
  - Chore: update kitsune2 dependencies to 0.4.0-dev.5
  - Chore: update Cargo.lock for kitsune2 0.4.0-dev.5
- Remove unused mock HDI (#5484) by @ThetaSinner
  - Chore: Remove unused mock HDI
  - Docs: Update changelog
  - Chore: update lock
  - Chore: Review changes
- Update project to use Rust 1.91 by @ThetaSinner
- Switch back to dev releases (#5459) by @ThetaSinner
- Prepare 0.6.0 release (#5453) by @ThetaSinner
- Switch to RC releases (#5432) by @ThetaSinner
  - Chore: Switch to RC releases
  - Chore: Permit RC releases
- Update to release version of Kitsune2 0.3.0 (#5429) by @ThetaSinner
  - Chore: Update to release version of Kitsune2 0.3.0
  - Docs: Update changelog
- Mark unpublished crates to make it clear to tooling that they aren't supposed to be published (#5324) by @ThetaSinner
  - Chore: Mark unpublished crates to make it clear to tooling that they aren't supposed to be published
  - Chore: Update cargo lock
  - Chore: Revert change to holochain_wasm_test_utils
- Upgrade Lair and Rust (#5317) by @ThetaSinner
  - Chore: Upgrade Lair and Rust
  - Docs: Update changelog
- Upgrade to `darling` v0.21 and `syn` v2.0 (#5275) by @ddd-mtl
- Upgrade `strum` and `strum_macros` to `0.27.x` (#5267) by @ddd-mtl
- One-off format of imports using unstable `group_imports` (#5239) by @ThetaSinner
- Update Kitsune2 to 0.2.15 and tx5 to 0.7.1 (#5211) by @ThetaSinner
- Remove links to the Holochain forum (deprecated) and tidy READMEs (#5115) by @ThetaSinner
- Fix some minor issues in the comments (#5068) by @shandongzhejiang
- Upgrade dependencies (#5059) by @ThetaSinner
- Update flakes, Rust version and split 0.5 version (#5039) by @ThetaSinner
  - Chore: Update flakes and split 0.5 version
  - Chore: Update workspace Rust version and fix clippy issues
  - Chore: Resolve clippy warnings about features
- Reference repository rather than homepage (#5040) by @ThetaSinner
- *(holochain)* Bump wasmer to v6 (#5001) by @jost-s
  - Chore(holochain): bump wasmer to v6
  - Ci: install llvm for llvm-objcopy for wasmer-wamr
  - Fix(hc_sandbox): await cmd of non running sandbox with output to prevent test from hanging
  - 
- Configuration schema for network config added to conductor schema (#4949) by @ThetaSinner
- Generate bundle schemas (#4935) by @ThetaSinner
  - Chore: Generate bundle schemas
  - Docs: Update changelog
  - Update crates/holochain_zome_types/Cargo.toml
- Filter local agents (#4914) by @ThetaSinner
  - Chore: Filter local agents
  - Chore: Update K2
  - Chore: Update to use new K2
  - Chore: Update to use new K2
  - Test: Only expect remote signals to others
  - Feat: Bridge calls and add tests
  - Chore: Bump k2
  - Chore: Bridge countersigning session negotiation
- Bump K2 0.1.2 (#4905) by @ThetaSinner
  - Chore: Bump K2 0.1.2
  - Chore: Implement op filter
- Remove all dpki related code (#4901) by @jost-s
  - Chore: remove all dpki related code
  - Chore: remove remnants of get_agent_key_lineage
  - Ci: move nextest installation to workflow to prevent complete rebuilds
  - Chore: remove all deepkey related code
  - Docs(changelog): mention dpki removal
  - Chore(holochain): remove revoke_agent_key call
  - Docs(changelog): mention removal of revoke_agent_key call
  - 
- Fix `mut self` warning (#4880) by @ThetaSinner
- Switch back to dev releases (#4883) by @ThetaSinner
  - Chore: Switch back to dev releases
  - Chore: Try pre minor
  - Chore: Match script
  - Chore: add debug
  - Chore: Update versions as though 0.5 had released
  - Chore: Format TOML
- Prepare for RC releases by @ThetaSinner
- Bump dependency versions (#4860) by @ThetaSinner
- Add missing backtick and remove redundant backtick (#4845) by @highcloudwind
- Enable and move hdk extern tests (#4799) by @c12i
  - Fix failing tests
  - Move tests
  - Remove cyclic dev dependency
- Remove redundant words in comment (#4529) by @guqicun
- Bump workspace Rust to latest stable (#4505) by @ThetaSinner
  - Chore: Bump workspace Rust to latest stable
  - Fix(kitsune_p2p): Invalid ordering strategy
  - Fixup! fix(kitsune_p2p): Invalid ordering strategy
- Remove deprecated code (#3885) by @jost-s
  - Chore: remove deprecated code  * docs: add deprecations to changelogs  * Update crates/holochain/CHANGELOG.md
- Fix typos (#3501) by @xiaoxianBoy
- Prepare 0.2.0 release and 0.3.0-beta-dev.N series (#2290) by @steveej
  - Chore: prepare 0.2.0 release and 0.3.0-beta-dev.N series  * docs(CHANGELOG): add toplevel release notes  * fixup! chore: prepare 0.2.0 release and 0.3.0-beta-dev.N series
- Change all version suffix to beta-rc (#2076) by @steveej
- Rename `holochain_deterministic_integrity` to `hdi` by @steveej
- *(CHANGELOGs)* Backfill for #1386 by @steveej

### Build System

- Add repository tooling and licences by @veeso
  - Cargo-make tasks, taplo config, pinned toolchain with the wasm32 target, Apache-2.0 and CAL-1.0 licence texts, README and changelog seed.
- Create the holochain-hdi workspace by @veeso
  - Crates moved from holochain/holochain (holochain/holochain#5400) now share one workspace version, managed by the holochain release integration.
- Remove `holochain_sqlite` crate (#5869) by @veeso
- Bump holochain_wasmer_* crates to 0.0.99 (#4632) by @mattyg
- \[**BREAKING**\] Update serialization to v0.0.54 (#3757) by @jost-s
  - Test: update conductor api serialization tests  * build: update holochain_serialized_bytes to v0.0.54  * build: bump serialization & wasmer  * build: bump wasmer  * build: bump holochain_wasmer_common  * build: bump rmp-serde  * test: adapt interface serialization tests  * docs: mention breaking change in changelog  * the fix the clip  * test: update serialized bytes test  * fix: network info request test  * update hc demo cli wasms

### CI

- Add test and release workflows by @veeso
  - Test workflow with a ci_pass aggregate job for the branch ruleset, and the holochain release integration prepare/publish workflows.

### Testing

- *(hdi/op)* Unignore and fix failing tests by @steveej
- Tests for countersigning validation by @thedavidmeister
- Test to show invalid mapping for validate callback by @thedavidmeister
- Test for must_get_x by @thedavidmeister
- Test for ephemeral sig by @thedavidmeister
- Tests for validation package callback by @thedavidmeister
- Tests for post commit by @thedavidmeister
- Tests for init callback by @thedavidmeister
- Tests compiling by @thedavidmeister

### Refactor

- Update cap model to make space for other grant types than zome calls and rename types to keep the set of types coherent (#5947) by @ThetaSinner
  - Refactor: Update cap model to make space for other grant types than zome calls and rename types to keep the set of types coherent.
  - Chore: review fixes
  - Chore: more review fixes
  - Chore: yet more review fixes
  - Chore: adjust the review fixes
  - Chore: condense capability grant changelog entries
  - Chore: guard
  - Fix: clippy ignore of an enum size difference by boxing
  - Chore: review changes
  - Chore: review changes
  - Docs: Mention `GrantZomeCallCapabilityGrant`
- *(hdi)* Give validate-callback FlatOp types precise per-variant action data (#5903) by @ThetaSinner
  - Introduce TypedAction<D>, pairing an action's ActionHeader with the exact ActionData payload already known from which FlatOp/OpEntry/OpUpdate/OpDelete/ OpRecord/OpActivity/OpLink variant it's in, instead of handing out the fully generic Action. This removes the unreachable!()-guarded re-matching that both hdi internals and downstream integrity zomes previously needed just to get back data the type already carried.
- Dissolve dht_v2 naming and remove migration-history language (#5881) by @ThetaSinner
  - Docs: remove stale legacy-vs-current comparison from state_model.md
  - Refactor(holochain): remove stale legacy-model comments and dead CellError variants
  - Refactor: remove stale legacy-model comments in holochain_data/conductor_api/state
  - Refactor(holochain_integrity_types): dissolve dht_v2 into action/op/record
  - Refactor(holochain_zome_types): dissolve dht_v2 into action/op/record
  - Refactor(holochain_types): unify dht_v2::WarrantOp into warrant::WarrantOp
  - Refactor(holochain_types): dissolve dht_v2 into op
  - Refactor(holochain_state,holochain_cascade): repoint dht_v2 references
  - Refactor(holochain_p2p,holochain_data): repoint dht_v2 references
  - Refactor(holochain): repoint dht_v2 references
  - Refactor(hdi): repoint dht_v2 references
  - Refactor(holochain_state): resolve the dht_v2-as-v2 alias in source_chain.rs
  - Refactor: repoint remaining dht_v2 references in holochain_p2p tests and holochain_data docs
  - Refactor: rename remaining _v2-suffixed identifiers and reword v2-model prose
  - Docs: add CHANGELOG entries for the _v2 identifier renames
  - Fix: resolve static-all fallout from dht_v2 dissolution
  - Remove a now-redundant explicit rustdoc link target on wire_rows_to_ops left over from the Task 13 rename.
  - Docs: note the hc binary build-order gap for hc_client/hc_sandbox tests
  - Docs: fix two more stale DB-split doc comments found in final review
  - Fixup! refactor: rename remaining _v2-suffixed identifiers and reword v2-model prose
  - Fixup! refactor(holochain_integrity_types): dissolve dht_v2 into action/op/record
  - Fixup! docs: add CHANGELOG entries for the _v2 identifier renames
  - Fix: validate the exact CreateLink hash in RegisterDeleteLink::new
- Reduce usage of DNA file and split inline zomes into a separate implementation (#5828) by @ThetaSinner
  - Refactor: Reduce usage of DNA file and split inline zomes into a separate implementation
  - Chore: Follow up
  - Chore: Follow up
  - Chore: static
  - Docs: Document inline-zome split and DNA-file API changes
  - Add an Unreleased changelog entry covering the breaking API changes (WasmZome -> WasmZomeDef, zome_hash, InlineZome::hash, ZomeDef::Inline holding an InlineHash, the DnaHash change from dropping the untagged serialization, removed DnaWithRole::replace_dna) and the additions (InlineHash/ZomeHash, DnaDef::replace_coordinators, ribosome restructure).
  - Also refresh the now-stale ZomeDef doc comment to match the new serialization and inline-zome identification.
  - Chore: review changes
  - Chore: review changes
  - Chore: fix feature logic
- \[**BREAKING**\] Run test workflow with iroh transport (#5552) by @jost-s
  - Feat: integrate iroh transport with iroh relay
  - Feat: add iroh transport targets to makefile
  - Fix: several tests in holochain and downstream crates
  - Test: increase consistency timeouts from 10 to 15 seconds
  - Chore: update kitsune2 to 0.4.0-dev.2
  - Ci: run test workflow with iroh transport
  - Docs: update changelog
  - Fix: app validation ops test
  - Fix: sandbox cli to use transport features
  - Refactor: parse signal and relay urls properly
- Replace `BTreeSet` with `HashSet` in `GrantedFunctions::Listed` (#5349) by @ThetaSinner
  - Refactor: Replace `BTreeSet` with `HashSet` in `GrantedFunctions::Listed`
  - Docs: Update changelog
  - Fixup! refactor: Replace `BTreeSet` with `HashSet` in `GrantedFunctions::Listed`
- Remove the unused NetId hash type from `holo_hash` (#5306) by @ThetaSinner
  - Refactor: Remove the unused NetIdHash type from `holo_hash`
  - Docs: Update changelog
  - Docs: Update changelog and readme
- \[**BREAKING**\] *(holochain)* Zome call signature verification (#4433) by @jost-s
  - Refactor!: add opaque field zome_call_payload to ZomeCall  * refactor(holochain): change zome call to signature and bytes  * refactor: rename ZomeCall to SignedZomeCall  * refactor: rename ZomeCallUnsigned to ZomeCallParams  * refactor: rename ZomeCallDeserialized to ZomeCall  * docs(holochain): metion zome call signing change in changelog  * refactor: rename SignedZomeCall to ZomeCallParamsSigned  * fix websocket interface test  * rename some variables  * refactor(holochain-types): delete unused module  * refactor(holochain-p2p): rename zome_call_payload to zome_call_params_serialized  * refactor: move ZomeCall to conductor crate  * refactor: remove unused method resign_zome_call  * fix imports  * fix: zomecall import  * yet another feature clippy fix  * refactor(holochain)!: sign hash of serialized bytes of zome call params  * refactor(holochain)!: sign hash of serialized bytes for remote signals  * refactor(holochain): dry hashing into serialization fn  * refactor: simplify serialized bytes and hash types  * refactor(holochain-p2p)!: pass params as serialized bytes into call_remote  * docs(holochain): explain in changelog how signature is verified  * refactor(holochain-p2p)!: simplify CallRemoteSignal  * style: fmt  * feat(holo-hash): add sha2 512-bit hasher  * refactor(holochain!:hash zome params to sign with sha2 512-bit  * docs(holochain): update changelog to reflect sha2 512-bit hasher  * fix(holochain): hash with sha2 512-bit to verify signature  * test(holochain): reintegrate and fix ser_regression_test  * style: fix clippiness  * docs(holochain-conductor-api): fix hash algorithm in CallZome  ---------
- Remove original action & entry from Op (#3627) by @jost-s
  - Refactor: remove original action & entry from RegisterUpdate  * refactor: remove original action & entry from RegisterDelete  * test: fix app validation main loop test  * refactor: simplify selection of zome to invoke for validation  * refactor: remove create link action from DhtOp::RegisterDeleteLink  * fix: create link wasm  * style: unnecessary clones  * ah jaysus why doesn't the ide tell me this earlier?!  * style: clippy fix  * Revert "refactor: remove create link action from DhtOp::RegisterDeleteLink"  This reverts commit c56eec8a0a2780135632bffda64a2835968c4571.  * test: add unit tests for get_zomes_to_invoke  * tests: split up into separate files  * test: add tests for update of update & delete of delete  * test: add tests for store entry op  * test: add tests for register update  * test: add tests for register delete  * test: add tests for register create & delete link  * test: add tests for store record + links  * fix thingies  * refactor: clean up get_zomes_to_invoke  * docs: update hdi & holochain_integrity_types changelogs  * test: set zome index to 0  * apply suggestion from code review
- Mock app validation network (#3545) by @jost-s
  - Test: remove test_ideas file from sys validation  * test: add validation callback test with must get action  * test: fix & extend main loop test  * revert Cargo.lock  * refactor: move op_to_record to zome_call_workflow  * test: add test if unresolved deps are fetched  * build: update cargo  * build: revert Cargo.lock  * build: revert to develop Cargo.lock  * test: update to multi-authored test space  * test: add test if unresolved agent activities are fetched  * revert dna_hash from ribosome in fn validate_op  * feat: prevent multiple fetches of identical hashes  * docs: update holochain changelog  * docs: revert auto-formatting with prettier  * refactor: fetch all missing hashes in one task  * refactor: return outcome summary to main workflow fn  * refactor: mock network in app validate op  * test: refactor test must_get_agent_activity to use mock network  * Apply suggestions from code review
- App validate ops with local data only (#3433) by @jost-s
  - Test: remove test_ideas file from sys validation  * test: add validation callback test with must get action  * test: fix & extend main loop test  * revert Cargo.lock  * refactor: move op_to_record to zome_call_workflow  * test: add test if unresolved deps are fetched  * build: update cargo  * build: revert Cargo.lock  * build: revert to develop Cargo.lock  * test: update to multi-authored test space  * test: add test if unresolved agent activities are fetched  * revert dna_hash from ribosome in fn validate_op  * feat: prevent multiple fetches of identical hashes  * docs: update holochain changelog  * docs: revert auto-formatting with prettier  * refactor: fetch all missing hashes in one task  * Apply suggestions from code review
- *(cargo)* Move some common deps into workspace (#3285) by @steveej
- *(zome-call)* Optimize cap grant verification (#2097) by @jost-s
  - Refactor(zome-call): cap grant verification  * refactor: simplify cap grant validation flow  * update changelog  * simplify assigned cap access arm  * add PR reference to changelog  * add comment to test  * Apply suggestions from code review
- *(generate-readme)* Switch to cargo-rdme (#1820) by @jost-s
  - Docs(hdi): prepare readme and lib for cargo-rdme  * docs(hdk): prepare lib and readme for cargo-rdme  * docs(holochain_keystore): prepare readme for cargo-rdme  * docs(holochain_state): prepare lib and readme for cargo-rdme  * docs(hdk+hdi): delete obsolete tpl files  * refactor(generate_readmes): replace cargo-readme with cargo-rdme  * fix(hdk): intra-doc link to hdk_extern macro  * ci(readme): remove cargo-readme and install cargo-rdme  * ci(create-readme): configure git  * build(generate-readme): unset git name & email afterwards  * docs(crate-level): generate readmes from doc comments  * fix clippy  ---------
- \[**BREAKING**\] *(uid)* Rename uid to network_seed (#1493) by @jost-s
  - Refactor(uid): rename uid to network_seed  * chore(all): update changelogs  * docs(dna): describe network seed  * docs(network_seed): add cross-links  * chore(changelogs): highlight breaking change

### Styling

- Adhere to clippy lints by @steveej

### Documentation

- *(crate-level)* Generate readmes from doc comments by @holochain-release-automation2
- *(crate-level)* Generate readmes from doc comments by @holochain-release-automation2
- *(crate-level)* Generate readmes from doc comments by @holochain-release-automation2
- *(hdi)* Fix doc-comment warnings in HDI crate (#4554) by @cdunster
  - Docs(hdi): fix doc-comment links in flat_op
  - Docs(hdi): fix doc-comment links in flat_op_record
  - Docs(hdi): fix doc-comment links in flat_op_entry
  - Docs(hdi): fix doc-comment links in flat_op_activity
  - Docs(hdi): fix doc-comment links in path module
  - Docs(hdi): fix doc-comment links in hash_path module
  - Docs(hdi): fix doc-comment links in x_salsa20_poly1305
  - Docs(hdi): fix doc-comment links in ed25519
  - Docs(hdi): fix doc-comment links in lib module
  - Docs(hdi): fix doc-comment links in hash_path shard module
  - Docs(hdi): fix doc-comment links in info module
  - Docs(hdi): add missing doc-comment links in entry module
  - Docs(hdi): add missing doc-comment links in path module
  - Docs(hdi): add missing doc-comment links in hash module
  - Docs(hdi): add missing doc-comment links in map_extern
  - Docs(hdi): add missing doc-comment links in lib module
  - Docs(hdi): add missing doc-comment links in hash_path
  - Docs(hdi): add more missing doc-comment links in entry module
  - Docs(hdi): fix typo in doc-comment in hash_path
- *(crate-level)* Generate readmes from doc comments by @holochain-release-automation2
- *(app-validation-workflow)* Add module documentation (#3794) by @jost-s
  - Docs(app-validation-wf): add module documentation  * docs: update changelog  * apply suggestions from code review  * docs: add section on validation errors  * update formatting  * apply suggestions from code review  * style: fix toml formatting
- Fix some rust doc links (#3552) by @jost-s
- *(crate-level)* Generate readmes from doc comments
- *(hdk)* Clean up and add example wasm links (#1771) by @jost-s
  - Add validation example to Op  * fix validation link in hdi  * add link to capability wasm example  * add example to emit_signal  * add some commas to entry mod  * link init callback to examples  * update changelog  * cargo fmt
- Docs by @thedavidmeister
- *(hdi)* Add docs and readme (#1532) by @freesig
  - Add docs and readme to hdi  * fix doctest  * add hdi to automatic README generation
- *(hdk)* Explain op type & how to get typed path (#1505) by @jost-s
  - Docs(hdk): reference hdi from integrity zome section  * docs(hdi): add a validation example to hdi docs  * docs(link): document get_links arg link_type  * docs(link): add same docs to get_link_details  * docs(link): add comments to LInkTypeFilterExt  * docs(link): add comments to LInkTypeFilterExt  * docs(hdk): intra-link to wasm error  * docs(derive): explain unit enum some more  * docs(get_link): filter only takes full range  * docs(link): clarify get links scope  * fmt  * docs(hdk): shot entry_defs cb as plain text  * docs(typed_link): add example how to get from path  * docs(validation): add op type example  * chore(changelogs): add docs  * add more explanatory text to OpType helper
- *(hdk/hdi)* Add more details to get_links (#1486) by @jost-s
  - Docs(hdk): reference hdi from integrity zome section  * docs(hdi): add a validation example to hdi docs  * docs(link): document get_links arg link_type  * docs(link): add same docs to get_link_details  * docs(link): add comments to LInkTypeFilterExt  * docs(link): add comments to LInkTypeFilterExt  * docs(hdk): intra-link to wasm error  * docs(derive): explain unit enum some more  * docs(get_link): filter only takes full range  * docs(link): clarify get links scope
- Docs by @freesig
- *(hdi)* Add crate level documentation (#1477) by @jost-s
  - Docs(hdk): reduce to only validate callback  * write create docs for holochain_deterministic_integrity  * flesh out doc comments  * add links to examples of integrity & ccordinator zomes  * fmt  * remove irrelevant phrase  * add dna manifest examples  * Update crates/holochain_deterministic_integrity/src/lib.rs
- *(hdi)* Add/edit crate level docs for hdi and hdk (#1443) by @jost-s
  - Docs(hdk): reduce to only validate callback  * write create docs for holochain_deterministic_integrity  * flesh out doc comments  * add links to examples of integrity & ccordinator zomes  * fmt  * remove irrelevant phrase  * add dna manifest examples  * Update crates/holochain_deterministic_integrity/src/lib.rs
- Docs to integrity by @freesig
- Docs by @thedavidmeister
- Docs by @neonphog

### Automated Changes

- Update dependabot.yml with shared content in [#2](https://github.com/holochain/holochain-hdi/pull/2)
- Update CODEOWNERS with shared content in [#1](https://github.com/holochain/holochain-hdi/pull/1)
- Merge pull request #1684 from holochain/release-20221130.011217 by @github-actions[bot]
- Merge pull request #1665 from holochain/release-20221123.011302 by @github-actions[bot]
- Merge pull request #1658 from holochain/release-20221116.012050 by @github-actions[bot]
- Merge pull request #1653 from holochain/release-20221109.012313 by @github-actions[bot]
- Merge pull request #1643 from holochain/release-20221102.014648 by @github-actions[bot]
- Merge pull request #1637 from holochain/release-20221026.192152 by @github-actions[bot]
- Merge pull request #1632 from holochain/release-20221019.014538 by @github-actions[bot]
- Merge pull request #1608 from holochain/release-20221005.164304 by @github-actions[bot]
- Merge pull request #1595 from holochain/release-20220928.014801 by @github-actions[bot]
- Merge pull request #1574 from holochain/release-20220914.013149 by @github-actions[bot]
- Merge pull request #1564 from holochain/release-20220908.155008 by @github-actions[bot]
- Merge pull request #1561 from holochain/release-20220907.100911 by @github-actions[bot]
- Merge pull request #1555 from holochain/release-20220907.014838 by @github-actions[bot]
- Merge pull request #1546 from holochain/release-20220831.015922 by @github-actions[bot]
- Merge pull request #1534 from holochain/release-20220820.111904 by @github-actions[bot]
- Merge pull request #1528 from holochain/release-20220817.013233 by @github-actions[bot]
- Merge pull request #1513 from holochain/release-20220810.012252 by @github-actions[bot]
- Merge pull request #1506 from holochain/release-20220803.124141 by @github-actions[bot]
- Merge pull request #1504 from holochain/release-20220728.122329 by @github-actions[bot]
- Merge pull request #1487 from holochain/release-20220713.013021 by @github-actions[bot]
- Merge pull request #1482 from holochain/release-20220710.155915 by @github-actions[bot]
- Merge pull request #1468 from holochain/release-20220701.181019 by @github-actions[bot]
- Merge pull request #1465 from holochain/release-20220629.012044 by @github-actions[bot]
- Merge pull request #1456 from holochain/release-20220622.133046 by @github-actions[bot]
- Merge pull request #1439 from holochain/release-20220616.084359 by @github-actions[bot]
- Merge pull request #1401 from holochain/release-20220601.012853 by @github-actions[bot]
- Merge pull request #1395 from holochain/release-20220525.012131 by @github-actions[bot]
- Merge pull request #1385 from holochain/release-20220518.010753 by @github-actions[bot]
- Merge pull request #1382 from holochain/release-20220511.012519 by @github-actions[bot]
- Merge pull request #1368 from holochain/release-20220429.205522 by @github-actions[bot]
- Merge pull request #1345 from holochain/release-20220414.075333 by @github-actions[bot]
- Merge pull request #1317 from holochain/release-20220330.010719 by @github-actions[bot]
- Merge pull request #1309 from holochain/release-20220323.023956 by @github-actions[bot]

### Other Changes

- Create a release from branch release-20260831.030531
- Create a release from branch release-20260820.161351
- Create a release from branch release-20260817.005712
- Create a release from branch release-20260803.022130
- Create a release from branch release-20260730.150702
- Create a release from branch release-20260729.163910
- Create a release from branch release-20260727.022800
- Create a release from branch release-20260721.103110
- Fix type name collisions and imports (#5899) by @ThetaSinner
- Create a release from branch release-20260716.145843
- Create a release from branch release-20260715.141229
- Unused deps and feature independence (#5868) by @ThetaSinner
- Complete switch to v2 model types (#5854) (#5860) by @ThetaSinner
- Create a release from branch release-20260701.171007
- Create a release from branch release-20260629.004342
- Create a release from branch release-20260525.004052
- Create a release from branch release-20260518.003858
- Create a release from branch release-20260504.004108
- Create a release from branch release-20260420.002833
- Refactor/must get agent activity logic (#5689) by @mattyg
- Create a release from branch release-20260330.002514
- Create a release from branch release-20260323.002355
- Create a release from branch release-20260316.002412
- Create a release from branch release-20260309.002048
- Create a release from branch release-20260302.002017
- Create a release from branch release-20260112.002450
- Add `GetStrategy` support to `Anchor` (#5548) by @ThetaSinner
- Create a release from branch release-20260105.001833
- Add GetStrategy configuration to TypedPath  (#5547) by @ThetaSinner
- Create a release from branch release-20251222.001720
- Feat/unify request timeout config (#5428) by @mattyg
- Create a release from branch release-20251201.001926
- Create a release from branch release-20251124.001659
- Create a release from branch release-20251119.115654
- Create a release from branch release-20251105.193427
- Create a release from branch release-20251105.001147
- Create a release from branch release-20251103.100646
- Create a release from branch release-20251029.001150
- Create a release from branch release-20251015.001251
- Create a release from branch release-20251008.002548
- Chore/rm unused types fns (#5325) by @mattyg
- Create a release from branch release-20251002.102845
- Create a release from branch release-20251001.001313
- Rustdoc formatting fixes (#5298) by @pdaoust
- Create a release from branch release-20250917.001123
- Fetch Reporting (#5274) by @neonphog
- Create a release from branch release-20250910.001116
- Push Release commits from release-20250827.001211 branch (#5248) by @cdunster
- Create a release from branch release-20250820.001144
- Create a release from branch release-20250806.001301
- Create a release from branch release-20250723.001249
- Create a release from branch release-20250716.001240
- Create a release from branch release-20250709.001228
- Create a release from branch release-20250702.001217
- Create a release from branch release-20250625.001242
- Create a release from branch release-20250611.001856
- Create a release from branch release-20250528.001153
- Re-enable network stats ci tests now that k2 is fixed (#5009) by @neonphog
- Add auth-material option to hc-demo-cli (#4994) by @neonphog
- Create a release from branch release-20250514.001133
- Authenticate (#4963) by @neonphog
- Create a release from branch release-20250430.001138
- Bump k2 to 0.2.2 (#4950) by @neonphog
- Create a release from branch release-20250423.001128
- Bump kitsune to include new tx5 which includes datachannel android vendor fix (#4919) by @neonphog
- Create a release from branch release-20250416.001135
- Create a release from branch release-20250404.120841
- Create a release from branch release-20250403.155950
- Create a release from branch release-20250219.004534
- Create a release from branch release-20250212.005142
- Chore/todo comments out of cargodocs (#4700) by @mattyg
- Change all enums exposed in the conductor API to use serde `tag = "type"` and `content = "value"` (#4616) by @matthme
- Separate kitsune_p2p_timestamp from holochain (#4686) by @ThetaSinner
- Create a release from branch release-20250205.005133
- Capability grants info for an app (#4647) by @nphias
- Create a release from branch release-20250129.004417
- Create a release from branch release-20250122.005106
- Remove arbitrary from tests (#4640) by @ThetaSinner
- Create a release from branch release-20250115.130248
- Zome calls hc sandbox (#4587) by @ThetaSinner
- Create a release from branch release-20250108.004547
- [Integrity_types] linktag conversion traits (#4574) by @nphias
- Create a release from branch release-20241225.004431
- Create a release from branch release-20241218.004735
- Create a release from branch release-20241211.005655
- Fix warnings from `proptest-derive` (#4520) by @ThetaSinner
- Create a release from branch release-20241204.005132
- Create a release from branch release-20241120.005400
- Create a release from branch release-20241113.005114
- Doc/dna properties macro (#4430) by @mattyg
- Put some hdk functions behind unstable flag (#4371) by @maackle
- Create a release from branch release-20241106.004411
- Fix bad Arbitrary impl for HoloHash (#4381) by @maackle
- HoloHash from functions avoid panicking unless desired (#4380) by @mattyg
- Create a release from branch release-20241016.174505
- Create a release from branch release-20241009.004404
- Prepare 0.5 dev (#4316) by @ThetaSinner
- Create a release from branch release-20240926.150323
- Create a release from branch release-20240925.004452
- Countersigning workflow (#4188) by @ThetaSinner
- Slim down release builds part 1 (#4282) by @ThetaSinner
- Prepare upgrade to rust 1.81 (#4275) by @ThetaSinner
- Create a release from branch release-20240918.004220
- Feat/wasmer interpreter integration (#4206) by @mattyg
- Create a release from branch release-20240904.004824
- Adapt Open and CloseChain to allow for Agent migration as well (#4142) by @maackle
- Get agent activity full (#4221) by @ThetaSinner
- Create a release from branch release-20240828.004055
- Create a release from branch release-20240823.100115
- Dynamic db encryption (#4198) by @neonphog
- Release 20240819.140303 (#4199) by @ThetaSinner
- Deepkey integration (#3463) by @maackle
- Create a release from branch release-20240717.004654
- Expose validation receipt info for actions (#4049) by @ThetaSinner
- Update Holochain version in Holonix for weekly (#4076) by @holochain-release-automation2
- Create a release from branch release-20240710.004628
- Delete all table data when cleaning up dangling cells (#4056) by @maackle
- Create a release from branch release-20240626.004529
- Create a release from branch release-20240619.004549
- Warrants pt 5: author ChainFork warrants (#3996) by @maackle
- Create a release from branch release-20240612.004512
- Clippy --all-targets (#3995) by @neonphog
- Warrants pt 3 -- authoring and integrating warrants (#3875) by @maackle
- Update dependencies (#3991) by @ThetaSinner
- Salvage cap grant test from #3859 (#3961) by @maackle
- Return size error before prefix error for hash deser (#3983) by @maackle
- Create a release from branch release-20240529.004721
- Create a release from branch release-20240523.125153
- Bump wasmer to 0.0.94 (#3907) by @ThetaSinner
- Update rust and remove crate2nix (#3857) by @ThetaSinner
- Warrants (so far) (#3865) by @maackle
- Create a release from branch release-20240515.004441
- Create a release from branch release-20240508.003809
- Make open/close HDK callable (#3804) by @ThetaSinner
- Create a release from branch release-20240501.152652
- Fixup bad 0.3.0 holochain release (#3799) by @ThetaSinner
- Create a release from branch release-20240501.004555
- Prepare for 0.4 dev releases (#3786) by @ThetaSinner
- Check toml formatting in ci (#3781) by @ThetaSinner
- Create a release from branch release-20240424.004413
- Dependency updates (#3657) by @ThetaSinner
- Create a release from branch release-20240417.004246
- Non-blocking wasm compilation (#3590) by @maackle
- App install timing investigation (#3567) by @maackle
- Create a release from branch release-20240410.004338
- Create a release from branch release-20240403.160133
- Fix upcoming lint issues (#3539) by @ThetaSinner
- Create a release from branch release-20240327.004300
- Revert workspace deps and touch all Cargo.toml to trigger re-dep (#3505) by @neonphog
- String conversions for LinkTag (#3268) by @ThetaSinner
- Create a release from branch release-20240320.003423
- [refactor] Refactor hdk_extern (#3441) by @c12i
- Create a release from branch release-20240313.004312
- Hdk functions for encryption / decryption by *signing* agent keypairs (#3412) by @neonphog
- Merge branch 'release-20240306.004209' into develop by @holochain-release-automation2
- Create a release from branch release-20240306.004209
- Make it possible to `cargo check --all-features` (#3392) by @maackle
- Create a release from branch release-20240228.004140
- Configure getrandom for WASM in the HDI (#3362) by @ThetaSinner
- Create a release from branch release-20240207.003254
- Tx5 ep3 integration of endpoint/connection state management refactor (#3287) by @neonphog
- Create a release from branch release-20240201.115513
- Bump Rust version (#3267) by @ThetaSinner
- Sneak in a changelog item (#3258) by @maackle
- Create a release from branch release-20240124.004605
- Create a release from branch release-20240117.004514
- Chore/rename hdk entry defs (#2979) by @mattyg
- Create a release from branch release-20240112.112002
- Sync main to develop 2024 01 11 (#3196) by @ThetaSinner
- Create a release from branch release-20240110.003625 (#3191) by @ThetaSinner
- Update copyright dates to 2024 (#3185) by @joshuavial
- Create a release from branch release-20231222.142916
- Merge branch 'develop' by @ThetaSinner
- Merge release branch release-20231213.003542 (#3165) by @ThetaSinner
- Move conductor services to own crate (#3097) by @maackle
- Bump wasmer to 0.0.91 (#3136) by @thedavidmeister
- Add a workspace-level dependency and get CI to accept it (#3098) by @maackle
- Create a release from branch release-20231213.003542
- Wasmer dir (#3025) by @thedavidmeister
- Hide Timestamp::now() from hdk (#3116) by @ThetaSinner
- Create a release from branch release-20231206.112234
- Feat/dna properties macro (#2880) by @mattyg
- Create a release from branch release-20231129.004341
- Refactor/move path to hdi (#2980) by @mattyg
- Create a release from branch release-20231122.004553
- Sys validation tests for `validate_op` (#3023) by @ThetaSinner
- Clean up mixed license usage (#2989) by @ThetaSinner
- Extract batch processing logic from the sys validation workflow (#2982) by @ThetaSinner
- Create a release from branch release-20231115.003452
- 2023 10 11 wasmer (#2951) by @thedavidmeister
- Create a release from branch release-20231101.003619
- Fix warnings (#2936) by @maackle
- Create a release from branch release-20231011.004956
- RoughInt prop test (#2874) by @maackle
- Refactor/avoid importing kitsune to client (#2857) by @mattyg
- Create a release from branch release-20231004.005318
- Add proptest::Arbitrary impls everywhere (#2727) by @maackle
- Bump to rust-1.71.1 (#2660) by @neonphog
- Bump holochain-serialization and holochain-wasmer deps (#2844) by @maackle
- Create a release from branch release-20230920.004520
- Create a release from branch release-20230913.003318
- Holo-ready CHC API updates (#2698) by @maackle
- Create a release from branch release-20230823.003418
- Pin serde (#2675) by @ThetaSinner
- Create a release from branch release-20230809.004243
- Create a release from branch release-20230726.004038
- Add more context to InvalidCommit errors on the source chain (#2591) by @ThetaSinner
- Make installed_hash optional in CloneOnly strategy (#2600) by @maackle
- Create a release from branch release-20230719.011122
- Update CHC API for Holo DO implementation (#2502) by @maackle
- Create a release from branch release-20230703.184956
- Create a release from branch release-20230621.004233
- Move entry def check to app validation and rewrite private entry check (#2059) by @maackle
- Wip on dna info 2 (#2366) by @thedavidmeister
- Create a release from branch release-20230614.004108
- Create a release from branch release-20230607.004739
- Create a release from branch release-20230531.004233
- Standardize debug hex (#2384) by @maackle
- Consistency improvements (#2423) by @maackle
- Create a release from branch release-20230524.003830
- Clamp all arcs to Empty or Full (#2352) by @maackle
- Create a release from branch release-20230427.171927
- Add missing TryFrom impls for holo hash conversions (#2283) by @maackle
- Create a release from branch release-20230426.003734
- Create a new hc command to run a holochain webrtc signal server (#2265) by @neonphog
- Preserialization dylib, Connor's work (#2218) by @maackle
- Create a release from branch release-20230420.162535
- Feature consistency for sqlite (#2248) by @ThetaSinner
- Create a release from branch release-20230412.003659
- Remove impossible HoloHash From conversions and make `HoloHash::retype()` private (#2191) by @maackle
- Create a release from branch release-20230405.003224
- AppManifest `version` becomes `installed_hash` (#2157) by @maackle
- Merge remote-tracking branch 'upstream/main' into pr_merge_main_into_develop by @steveej
- Create a release from branch release-20230322.003727
- Improve HoloHash::from_raw_32 and use it (#2162) by @maackle
- Fix documentation typos (#2142) by @ThetaSinner
- Create a release from branch release-20230315.183209
- Tracing Fixes (#2079) by @neonphog
- Deepkey integration pt 1. - Deepkey Conductor Service (#2024) by @maackle
- OpType -> FlatOp + some reorg (#1909) by @maackle
- Create a release from branch release-20230126.223635
- Create a release from branch release-20230120.225800
- 2023 01 19 wasmer version (#1781) by @thedavidmeister
- Update copyright year 2023 (#1740) by @Seb33300
- Create a release from branch release-20230117.165308
- Make the two entry limits match at 4MB (#1762) by @maackle
- Rollup of rust related PRs (#1735) by @steveej
- Create a release from branch release-20221223.034701
- Create a release from branch release-20221215.173657
- Apply develop versions to changed crates
- Improvements to to_type for ergonomic validation (#1702) by @zippy
- Bump one_err + sodoken + lair (#1718) by @neonphog
- Create a release from branch release-20221130.011217
- Apply develop versions to changed crates
- Merge pull request #1670 from holochain/secure-primitive-dedupe by @maackle
- Remove duplicate secure_primitives! macro by @maackle
- Merge pull request #1667 from holochain/2022-11-23-ergo by @thedavidmeister
- Fmt by @thedavidmeister
- Rename more stuff by @thedavidmeister
- Wip on renaming by @thedavidmeister
- Renaming zome id to zome index by @thedavidmeister
- Rename ZomeId to ZomeIndex by @thedavidmeister
- Create a release from branch release-20221123.011302
- Apply develop versions to changed crates
- Create a release from branch release-20221116.012050
- Apply develop versions to changed crates
- Merge pull request #1654 from holochain/bump_some_deps_22_11_09 by @neonphog
- Bump dependencies by @robbiecarlton
- Create a release from branch release-20221109.012313
- Apply develop versions to changed crates
- Merge pull request #1620 from holochain/2022-10-10-wasmer by @thedavidmeister
- Lint by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-10-10-wasmer by @thedavidmeister
- Create a release from branch release-20221102.014648
- Apply develop versions to changed crates
- Create a release from branch release-20221026.192152
- Apply develop versions to changed crates
- Merge pull request #1609 from holochain/kitsune-diagnostics by @maackle
- Merge pull request #1610 from holochain/more-metrics by @maackle
- Merge pull request #1633 from holochain/diagnostic-groundwork by @maackle
- Many small changes made during the course of diagnostic testing spike by @maackle
- Create a release from branch release-20221019.014538
- Apply develop versions to changed crates
- Merge pull request #1622 from holochain/remove-dylib-config by @maackle
- Remove dylib config in Cargo.tomls by @maackle
- Bump wasmer by @thedavidmeister
- Create a release from branch release-20221005.164304
- Apply develop versions to changed crates
- Merge pull request #1601 from holochain/fix-holohash-from by @maackle
- Clippies by @maackle
- Make OpBasis AnyLinkableHash instead of AnyDhtHash by @maackle
- Create a release from branch release-20220928.014801
- Apply develop versions to changed crates
- Merge pull request #1489 from holochain/chc by @maackle
- Merge remote-tracking branch 'origin/develop' into chc by @maackle
- Create a release from branch release-20220914.013149
- Apply develop versions to changed crates
- Merge remote-tracking branch 'origin/develop' into chc by @maackle
- Merge pull request #1571 from holochain/fix-lair-config by @neonphog
- Merge branch 'develop' into fix-lair-config by @neonphog
- Checkpoint by @neonphog
- CHC implementation by @maackle
- Merge pull request #1538 from holochain/chain-graft by @maackle
- Merge remote-tracking branch 'origin/develop' into chain-graft by @maackle
- Merge pull request #1566 from holochain/remove-validation-package by @maackle
- Merge remote-tracking branch 'origin/develop' into remove-validation-package by @maackle
- Remove all notion of validation package by @maackle
- Merge remote-tracking branch 'origin/develop' into chain-graft by @maackle
- Merge pull request #1509 from holochain/cascade-readthrough by @maackle
- Merge remote-tracking branch 'origin/develop' into cascade-readthrough by @maackle
- Fix hdi integration tests by @maackle
- Merge remote-tracking branch 'origin/develop' into chain-graft by @maackle
- Create a release from branch release-20220908.155008
- Apply develop versions to changed crates
- Merge pull request #1542 from holochain/large-entry-gossip-test by @maackle
- Merge remote-tracking branch 'origin/develop' into large-entry-gossip-test by @maackle
- Create a release from branch release-20220907.100911
- Apply develop versions to changed crates
- Merge pull request #1559 from holochain/pr-hdi-0.1-rollup by @steveej
- Minor adjustments by @maackle
- Merge remote-tracking branch 'origin/develop' into large-entry-gossip-test by @maackle
- Create a release from branch release-20220907.014838
- Apply develop versions to changed crates
- Merge remote-tracking branch 'origin/develop' into large-entry-gossip-test by @maackle
- Merge pull request #1543 from holochain/fix-bandwidth-throttles by @maackle
- Merge remote-tracking branch 'origin/develop' into fix-bandwidth-throttles by @maackle
- Merge pull request #1501 from holochain/chain-item-generic by @maackle
- Merge remote-tracking branch 'origin/chain-item-generic' into large-entry-gossip-test by @maackle
- Merge pull request #1524 from holochain/chain-item-generic-isotest by @maackle
- Merge remote-tracking branch 'origin/develop' into chain-item-generic-isotest by @maackle
- Merge pull request #1450 from holochain/publicize-validation-types by @abe-njama
- Merge branch 'develop' of https://github.com/holochain/holochain into publicize-validation-types by @abe-njama
- Merge pull request #1452 from holochain/pub-hdi-crate by @thedavidmeister
- Merge branch 'develop' of https://github.com/holochain/holochain into pub-hdi-crate by @abe-njama
- Expose all validation types as public in holochain_deterministic_integrity by @abe-njama
- Every type that is in these crates is changed to fully public as they are serialized (and therefore public anyway). by @abe-njama
- Merge branch 'chain-item-generic' into chain-item-generic-isotest by @maackle
- Merge remote-tracking branch 'origin/develop' into chain-item-generic by @maackle
- Create a release from branch release-20220831.015922
- Apply develop versions to changed crates
- Merge pull request #1530 from holochain/must-get-agent-activity-cache-entry by @thedavidmeister
- Merge branch 'develop' into must-get-agent-activity-cache-entry by @thedavidmeister
- Merge pull request #1502 from holochain/must-get-agent-activity-host-fn by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into must-get-agent-activity-host-fn by @thedavidmeister
- Add entry def to cache entry by @freesig
- Add cached entry to chain filter by @freesig
- Add optional entry to register agent activity op by @freesig
- Pr fixes by @freesig
- Changelogs by @freesig
- Clippy by @freesig
- Working must_get_agent_activity by @freesig
- Add interface and coacade test by @freesig
- WIP by @maackle
- Rearrange initiate logic, better Debug impl for AppEntryBytes, fail test by @maackle
- Add large entry gossip test (with entries that are not very large yet) by @maackle
- Merge branch 'chain-item-generic' into chain-item-generic-isotest by @maackle
- Merge remote-tracking branch 'origin/develop' into chain-item-generic by @maackle
- Merge branch 'chain-item-generic' into chain-item-generic-isotest by @maackle
- Fix hash construction discrepancy, with errors to fix later by @maackle
- Merge remote-tracking branch 'origin/develop' into chain-item-generic by @maackle
- Merge branch 'chain_head_db_option' into chain-graft by @maackle
- WIP graft chain by @maackle
- Merge branch 'chain-query-order' into chain-graft by @maackle
- Create a release from branch release-20220820.111904
- Apply develop versions to changed crates
- Merge pull request #1514 from holochain/test-wasm-memory by @thedavidmeister
- Merge branch 'develop' into test-wasm-memory by @thedavidmeister
- Merge branch 'develop' into test-wasm-memory by @thedavidmeister
- Rollback wasmer by @thedavidmeister
- Wip on debugging wasm by @thedavidmeister
- ChainGraft for more nuanced record insertion by @maackle
- Merge remote-tracking branch 'origin/develop' into chain-item-generic-isotest by @maackle
- Merge pull request #1508 from holochain/demonstrate-ci-problem by @maackle
- Merge branch 'develop' into demonstrate-ci-problem by @maackle
- Merge pull request #1523 from holochain/crate-ver-info by @neonphog
- Merge branch 'crate-ver-info' of github.com:holochain/holochain into crate-ver-info by @neonphog
- Merge branch 'develop' into crate-ver-info by @neonphog
- Merge branch 'develop' into crate-ver-info by @neonphog
- Add hdi_version_req to BUILD_INFO by @neonphog
- Fix another test by @maackle
- Ignore failing tests by @maackle
- Remove panicking test by @maackle
- Add a panic to show that CI is not running these tests by @maackle
- Create a release from branch release-20220817.013233
- Apply develop versions to changed crates
- Merge pull request #1483 from holochain/must-get-agent-activity by @freesig
- Merge branch 'develop' into must-get-agent-activity by @freesig
- Merge pull request #1515 from holochain/new-lair-hc-sandbox by @neonphog
- DRY passphrase fetch by @neonphog
- Create a release from branch release-20220810.012252
- Apply develop versions to changed crates
- Merge branch 'develop' of github.com:holochain/holochain into must-get-agent-activity by @freesig
- Pr fixes by @freesig
- Merge branch 'develop' into must-get-agent-activity by @freesig
- Merge branch 'add-chain-filter' into must-get-agent-activity by @freesig
- Merge branch 'add-chain-filter' into must-get-agent-activity by @freesig
- Merge branch 'add-chain-filter' of github.com:/holochain/holochain into must-get-agent-activity by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into must-get-agent-activity by @freesig
- Add docs and comments by @freesig
- WIP by @freesig
- Fmt by @maackle
- Use isotest by @maackle
- Move all ChainItem stuff into holochain_types by @maackle
- Re-add chain_to_ops by @maackle
- Remove extraneous Deref by @maackle
- Fmt by @maackle
- Rewrite chain filter tests in terms of TestChainItem by @maackle
- Add ChainItem abstraction and use it for chain validation by @maackle
- Merge pull request #1507 from holochain/fix-hdi-tests by @maackle
- Fix hdi integration tests by @maackle
- Create a release from branch release-20220803.124141
- Apply develop versions to changed crates
- Create a release from branch release-20220728.122329
- Apply develop versions to changed crates
- Merge pull request #1495 from holochain/hex-debug by @maackle
- Merge branch 'develop' into hex-debug by @maackle
- Merge pull request #1463 from holochain/add-chain-filter by @freesig
- Pr fixes by @freesig
- Pull changes upstream by @freesig
- Merge branch 'develop' into add-chain-filter by @freesig
- Add ChainFilter and ChainFilterIter by @freesig
- Fix suggestion by @maackle
- Apply suggestions from code review by @maackle
- Use const time hex encoding for secure_primitive Debug by @maackle
- Merge pull request #1488 from holochain/validation-match-helpers by @freesig
- Changelogs by @freesig
- Merge branch 'validation-match-helpers' of github.com:/holochain/holochain into validation-match-helpers by @freesig
- Merge branch 'develop' into validation-match-helpers by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into validation-match-helpers by @freesig
- Pr fixes by @freesig
- Clippy by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into validation-match-helpers by @freesig
- Merge pull request #1402 from holochain/fix-query by @maackle
- Merge remote-tracking branch 'origin/develop' into fix-query by @maackle
- Create a release from branch release-20220713.013021
- Apply develop versions to changed crates
- Merge remote-tracking branch 'origin/develop' into fix-query by @maackle
- Fix query for simple seq range. Still broken for other ranges. by @maackle
- More testing by @freesig
- Merge branch 'validation-match-helpers' of github.com:/holochain/holochain into validation-match-helpers by @freesig
- WIP by @freesig
- WIP by @freesig
- WIP by @freesig
- WIP by @freesig
- Extract ops to structs by @freesig
- Merge pull request #1473 from holochain/rename_hdi by @steveej
- Create a release from branch release-20220710.155915
- Apply develop versions to changed crates
- Create a release from branch release-20220701.181019
- Apply develop versions to changed crates
- Merge pull request #1466 from holochain/quantized-gossip by @maackle
- Merge remote-tracking branch 'origin/develop' into quantized-gossip by @maackle
- Create a release from branch release-20220629.012044
- Apply develop versions to changed crates
- Merge pull request #1453 from holochain/add-zome-id by @freesig
- Changelogs by @freesig
- Changelogs by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into add-zome-id by @freesig
- Simplify function by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into add-zome-id by @freesig
- Make scoped zome types easier to use by @freesig
- Add zome id back into headers by @freesig
- Merge pull request #1459 from holochain/quantized-gossip-5 by @maackle
- Merge remote-tracking branch 'origin/develop' into HEAD by @maackle
- Merge pull request #1446 from holochain/integrate-shared-secret by @neonphog
- Merge branch 'develop' into integrate-shared-secret by @neonphog
- Merge pull request #1430 from holochain/weight-hdk01 by @maackle
- CHANGELOG by @maackle
- Merge branch 'develop' into weight-hdk01 by @maackle
- Create a release from branch release-20220622.133046
- Apply develop versions to changed crates
- Merge remote-tracking branch 'origin/develop' into weight-hdk01 by @maackle
- Merge branch 'weight-1' into weight-hdk01 by @maackle
- Grammar by @maackle
- Grammar by @maackle
- Merge branch 'weight-1' into weight-hdk01 by @maackle
- Merge remote-tracking branch 'origin/develop' into weight-1 by @maackle
- Merge branch 'weight-1' into weight-hdk01 by @maackle
- Merge branch 'weight' into weight-1 by @maackle
- Remove some cruft by @maackle
- Merge branch 'weight' into weight-1 by @maackle
- Proper hookup of countersigning weight info by @maackle
- Some comments by @maackle
- Add Rate types. Make weighted and unweighted header types. by @maackle
- Merge branch 'develop' into integrate-shared-secret by @neonphog
- Merge branch 'rand-update' into integrate-shared-secret by @neonphog
- Checkpoint shared secret by @neonphog
- Merge remote-tracking branch 'origin/develop' into quantized-gossip-5 by @maackle
- Merge pull request #1394 from holochain/2022-05-23-countersigning by @thedavidmeister
- Merge branch 'develop' into 2022-05-23-countersigning by @thedavidmeister
- Merge pull request #1445 from holochain/rand-update by @neonphog
- Merge branch 'develop' into rand-update by @neonphog
- Bump rand version && required associated fixes by @neonphog
- Lint by @thedavidmeister
- Changelog by @thedavidmeister
- Lint by @thedavidmeister
- Lint by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-05-23-countersigning by @thedavidmeister
- Merge pull request #1441 from holochain/record-action-entry by @maackle
- Grammar by @maackle
- Element -> Record by @maackle
- Fix some capitalization problems by @maackle
- Grammar by @maackle
- Header -> Action by @maackle
- Merge branch 'develop' of github.com:holochain/holochain into 2022-05-23-countersigning by @thedavidmeister
- Create a release from branch release-20220616.084359
- Apply develop versions to changed crates
- Merge pull request #1436 from holochain/pr_backfill_changelog_pr1386 by @steveej
- Merge pull request #1386 from holochain/2022-05-17-wasm-metering by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-05-17-wasm-metering by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-05-17-wasm-metering by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-05-23-countersigning by @thedavidmeister
- Merge pull request #1434 from holochain/integrity-dna-def-changelog by @freesig
- Changelog for holochain types by @freesig
- Changelog for hdi by @freesig
- Changelog for integrity types by @freesig
- Merge pull request #1325 from holochain/integrity-dna-def by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into integrity-dna-def by @freesig
- Merge pull request #1410 from holochain/hdk-shared-secret by @neonphog
- Changelog by @neonphog
- Merge branch 'develop' into hdk-shared-secret by @neonphog
- Use bytes for key ref by @neonphog
- Hdk shared secret api by @neonphog
- Update error and handle out of range u8 by @freesig
- Renames by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into integrity-dna-def by @freesig
- Merge pull request #1380 from holochain/2022-05-04-wasmer-bump by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-05-04-wasmer-bump by @thedavidmeister
- Change yaml layout by @freesig
- Add try from EntryDefIndex for UnitEntryTypes by @freesig
- Comment in ribosome by @freesig
- Documentation by @freesig
- Add examples to hdi by @freesig
- Merge branch 'develop' of github.com:holochain/holochain into integrity-dna-def by @freesig
- Document derives by @freesig
- Wip by @freesig
- Document hdk by @freesig
- Document HDI types and holochain_zome_types by @freesig
- Document integrity types by @freesig
- Add docs and remove require_validation_type by @freesig
- Remove unused types by @freesig
- Clippy by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into integrity-dna-def by @freesig
- Create a release from branch release-20220601.012853
- Apply develop versions to changed crates
- Merge pull request #1400 from Seb33300/develop by @steveej
- Update copyright year by @Seb33300
- Create a release from branch release-20220525.012131
- Apply develop versions to changed crates
- Merge pull request #1389 from holochain/fix-trace-feature-again by @freesig
- Merge branch 'develop' into fix-trace-feature-again by @freesig
- Changelog by @freesig
- Actually test tracing is working and fix feature flag by @freesig
- WIP by @freesig
- WIP by @freesig
- WIP by @freesig
- WIP by @freesig
- WIP by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into integrity-dna-def by @freesig
- WIP by @freesig
- Wip on countersigning m of n by @thedavidmeister
- Validation for m of n preflight by @thedavidmeister
- Wip on countersigning integrity checking by @thedavidmeister
- Basic metering by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-05-04-wasmer-bump by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-05-04-wasmer-bump by @thedavidmeister
- Fmt by @thedavidmeister
- Compiling wasmer bump by @thedavidmeister
- Merge branch 'quantized-gossip-2' into quantized-gossip-3 by @maackle
- Merge remote-tracking branch 'origin/quantized-gossip' into quantized-gossip-2 by @maackle
- Merge pull request #1383 from holochain/pr_rust_1_60 by @steveej
- Create a release from branch release-20220518.010753
- Apply develop versions to changed crates
- Merge branch 'quantized-gossip-2' into quantized-gossip-3 by @maackle
- Merge remote-tracking branch 'origin/develop' into quantized-gossip-2 by @maackle
- Merge pull request #1384 from holochain/remove-dev-dep-versions by @maackle
- Remove all versions from all dev deps by @maackle
- Merge branch 'quantized-gossip-2' into quantized-gossip-3 by @maackle
- Merge remote-tracking branch 'origin/develop' into quantized-gossip-2 by @maackle
- Merge remote-tracking branch 'origin/develop' into quantized-gossip-3 by @maackle
- Create a release from branch release-20220511.012519
- Apply develop versions to changed crates
- Revert "Revert all changes since 3e27aa0ad0c, except to dht crate" by @maackle
- Revert all changes since 3e27aa0ad0c, except to dht crate by @maackle
- Merge branch 'quantized-gossip-1' into quantized-gossip by @maackle
- Merge pull request #1365 from holochain/hdk-ergonomics by @maackle
- More info in the changelog by @maackle
- Add prism-like methods to convert composite hashes to their runtime primitives by @maackle
- Merge remote-tracking branch 'origin/develop' into hdk-ergonomics by @maackle
- Create a release from branch release-20220429.205522
- Apply develop versions to changed crates
- Better macro error for hdk_extern by @maackle
- Use Into for hdk link hash params by @maackle
- Merge remote-tracking branch 'origin/develop' into quantized-gossip by @maackle
- Merge pull request #1348 from holochain/release-20220421.145237 by @neonphog
- Merge remote-tracking branch 'upstream/develop' into release-20220421.145237 by @steveej
- Merge pull request #1323 from holochain/docs/fix-item-links by @jost-s
- Merge branch 'develop' of https://github.com/holochain/holochain into docs/fix-item-links by @jost-s
- Merge branch 'develop' of https://github.com/holochain/holochain into docs/fix-item-links by @jost-s
- Update all changelogs by @jost-s
- Create a release from branch release-20220421.145237
- Apply develop versions to changed crates
- Merge branch 'quantized-gossip-1' into quantized-gossip by @maackle
- Merge pull request #1335 from holochain/fix-trace-feature by @maackle
- Fix error msg by @maackle
- Merge branch 'develop' into fix-trace-feature by @maackle
- Update crates/holochain_deterministic_integrity/src/hdi.rs by @maackle
- Merge remote-tracking branch 'origin/fix-trace-feature' into quantized-gossip by @maackle
- Fixes bad trace feature by @freesig
- Merge remote-tracking branch 'origin/develop' into quantized-gossip by @maackle
- Create a release from branch release-20220414.075333
- Apply develop versions to changed crates
- Merge pull request #1342 from holochain/pr_cargo_manifest_keyword_check by @steveej
- Don't exceed manifest keywords char-limit
- Merge pull request #1341 from holochain/pr_merge_main_to_develop by @steveej
- Create a release from branch release-20220413.011152
- Apply develop versions to changed crates
- Merge pull request #1324 from holochain/integrity by @freesig
- Add changelogs by @freesig
- Change idk to hdi by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into integrity by @freesig
- Merge pull request #1329 from holochain/main by @steveej
- Create a release from branch release-20220406.010602
- Apply develop versions to changed crates
- Merge pull request #1308 from holochain/2022-03-21-compound-external by @thedavidmeister
- Merge branch 'develop' into 2022-03-21-compound-external by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-03-21-compound-external by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-03-21-compound-external by @thedavidmeister
- Wip on linkable hash by @thedavidmeister
- Wip on AnyLinkableHash by @thedavidmeister
- Update docs mock docs by @freesig
- Update docs by @freesig
- Change genesis self check to take dna info so it can run on the integrity zome by @freesig
- WIP by @freesig
- Merge pull request #1305 from holochain/stable-val-crates by @freesig
- Merge branch 'develop' into stable-val-crates by @freesig
- Merge pull request #1298 from holochain/2022-03-17-external-hash by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-03-17-external-hash by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-03-17-external-hash by @thedavidmeister
- Remove External from HDK HashInput by @thedavidmeister
- Add external hash type by @thedavidmeister
- Add changelog to integrity types by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into stable-val-crates by @thedavidmeister
- Create a release from branch release-20220330.010719
- Apply develop versions to changed crates
- Move zome and links by @freesig
- Test shows failure in fetching the ops for the regions that were created by @maackle
- Merge pull request #1283 from holochain/unidirectional-arcs-2 by @maackle
- Merge remote-tracking branch 'origin/develop' into unidirectional-arcs-2 by @maackle
- Create a release from branch release-20220323.023956
- Apply develop versions to changed crates
- Merge pull request #1284 from holochain/docs/fix-item-links by @jost-s
- Merge branch 'develop' of https://github.com/holochain/holochain into docs/fix-item-links by @jost-s
- Merge pull request #1292 from holochain/pr_merge_main_to_develop_again by @steveej
- Merge remote-tracking branch 'upstream/main' into pr_merge_main_to_develop_again
- Create a release from branch release-20220316.022611
- Apply develop versions to changed crates
- Merge release-20220316.022611 back into develop (#1289) by @holochain-release-automation2
- Clippy by @maackle
- Merge pull request #1272 from holochain/release-20220303.215755 by @steveej
- Create a release from branch release-20220303.215755
- Apply develop versions to changed crates
- Merge pull request #1253 from holochain/rust-2021 by @steveej
- Merge branch 'develop' into rust-2021 by @maackle
- Merge pull request #1255 from holochain/release-20220223.090000 by @steveej
- Create a release from branch release-20220223.090000
- Apply develop versions to changed crates
- Update Rust edition to 2021 by @maackle
- Merge pull request #1230 from holochain/2022-02-14-keccak256 by @thedavidmeister
- Merge branch 'develop' into 2022-02-14-keccak256 by @thedavidmeister
- Merge pull request #1228 from holochain/2022-02-11-blake2b by @thedavidmeister
- Merge branch '2022-02-11-blake2b' of github.com:holochain/holochain into 2022-02-14-keccak256 by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-02-11-blake2b by @thedavidmeister
- Merge pull request #1237 from holochain/default-sharding-2 by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into default-sharding-2 by @freesig
- Merge pull request #1211 from holochain/signed-hashed by @freesig
- Changelog by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into signed-hashed by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into signed-hashed by @freesig
- Pr fixes by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into develop by @freesig
- Merge pull request #1186 from holochain/init-can-call by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into init-can-call by @freesig
- Merge branch 'init-can-call' of github.com:/holochain/holochain into init-can-call by @freesig
- Lint by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2022-02-11-blake2b by @thedavidmeister
- Merge pull request #1229 from holochain/release-20220211.091841 by @steveej
- Create a release from branch release-20220211.091841
- Apply develop versions to changed crates
- Keccak and sha3 by @thedavidmeister
- Blake2b support by @thedavidmeister
- Merge pull request #1200 from holochain/flaky by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into flaky by @freesig
- Merge pull request #1207 from holochain/release-20220202.112225 by @steveej
- Create a release from branch release-20220202.112225
- Apply develop versions to changed crates
- Comments by @freesig
- Merge pull request #1196 from holochain/release-20220126.200716 by @steveej
- Create a release from branch release-20220126.200716
- Merge pull request #1193 from holochain/release-20220120.093525 by @jost-s
- Create a release from branch release-20220120.093525
- Merge pull request #1179 from holochain/release-20220106.093622 by @jost-s
- Create a release from branch release-20220106.093622
- Apply develop versions to changed crates
- Merge pull request #1167 from holochain/faster-validation by @freesig
- Merge branch 'develop' into faster-validation by @freesig
- Merge pull request #1171 from holochain/release-20211222.094252 by @steveej
- Create a release from branch release-20211222.094252
- Merge pull request #1162 from holochain/docs-holohashb64 by @maackle
- Indicate in HoloHashB64 docs that Serialize is the disinction by @timotree3
- Merge pull request #1159 from holochain/release-20211208.091009 by @steveej
- Create a release from branch release-20211208.091009
- Apply develop versions to changed crates
- Merge pull request #1152 from holochain/todo-sweep-maackle by @maackle
- Todo sweep of entire `holochain` crate by @maackle by @maackle
- Merge pull request #1140 from holochain/release-20211124.093220 by @steveej
- Create a release from branch release-20211124.093220
- Apply develop versions to changed crates
- Merge pull request #1130 from holochain/db-refactor-final by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into db-refactor-final by @freesig
- Merge pull request #1133 from holochain/release-20211117.094411 by @steveej
- Create a release from branch release-20211117.094411
- Merge pull request #1117 from holochain/release-20211110.083530 by @steveej
- Merge remote-tracking branch 'upstream/develop' into release-20211110.083530 by @steveej
- Create a release from branch release-20211110.083530
- Apply develop versions to changed crates
- All db refactor changes by @freesig
- Merge pull request #1116 from holochain/remove-unused-deps by @freesig
- Remove unused deps to speed up compile times by @freesig
- Merge pull request #1088 from holochain/release-20211103.094627 by @steveej
- Create a release from branch release-20211103.094627
- Apply develop versions to changed crates
- Merge pull request #1074 from holochain/release-20211027.100746 by @steveej
- Create a release from branch release-20211027.100746
- Apply develop versions to changed crates
- Merge pull request #1062 from holochain/release-20211021.140006 by @steveej
- Merge remote-tracking branch 'upstream/develop' into release-20211021.140006 by @steveej
- Merge pull request #1049 from holochain/2021-10-17-verbose-type by @thedavidmeister
- Merge branch 'develop' into 2021-10-17-verbose-type by @thedavidmeister
- Update changelogs by @thedavidmeister
- Remove verbose types by @thedavidmeister
- Create a release from branch release-20211021.140006
- Apply develop versions to changed crates
- Merge pull request #1058 from holochain/release-20211020.171211 by @steveej
- Create a release from branch release-20211020.171211
- Apply develop versions to changed crates
- Merge pull request #1000 from holochain/2021-09-21-post-commit by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2021-09-21-post-commit by @thedavidmeister
- Merge pull request #1040 from holochain/new-lair-3 by @neonphog
- Merge branch 'develop' into new-lair-3 by @neonphog
- Merge pull request #1028 from holochain/pr/rust-1.55.0 by @steveej
- Merge branch 'develop' into new-lair-3 by @neonphog
- Merge branch 'develop' into new-lair-3 by @neonphog
- Checkpoint normalizing test keystores by @neonphog
- Merge branch 'develop' of github.com:holochain/holochain into 2021-09-21-post-commit by @thedavidmeister
- Merge pull request #1043 from holochain/release-20211013.091723 by @steveej
- Create a release from branch release-20211013.091723
- Apply develop versions to changed crates
- Merge branch 'develop' of github.com:holochain/holochain into 2021-09-21-post-commit by @thedavidmeister
- Merge pull request #1034 from holochain/mock-network-kitsune by @freesig
- Changes to kitsune to allow network mocking by @freesig
- Merge branch 'develop' into 2021-09-21-post-commit by @thedavidmeister
- Merge pull request #1019 from holochain/dht-location-refactor by @maackle
- Bump version by @maackle
- Merge remote-tracking branch 'origin/develop' into dht-location-refactor by @maackle
- Merge branch 'develop' into dht-location-refactor by @neonphog
- Add ToSql for DhtLocation by @maackle
- Deeper DhtLocation integration by @maackle
- Merge branch 'develop' of github.com:holochain/holochain into 2021-09-21-post-commit by @thedavidmeister
- Merge pull request #789 from holochain/holo-hash-prefixes by @neonphog
- Merge branch 'develop' into holo-hash-prefixes by @neonphog
- Merge pull request #1026 from holochain/release-20211006.105406 by @steveej
- Create a release from branch release-20211006.105406
- Apply develop versions to changed crates
- Merge pull request #1023 from holochain/bump-rusqlite by @neonphog
- Bump rusqlite to 0.26.0 by @neonphog
- Update crates/holo_hash/src/hash_type/primitive.rs by @neonphog
- Merge branch 'develop' into holo-hash-prefixes by @neonphog
- Add HoloHost hash types by @zippy
- Merge branch 'develop' into 2021-09-21-post-commit by @thedavidmeister
- Merge pull request #1011 from holochain/release-20210929.090317 by @steveej
- Create a release from branch release-20210929.090317
- Merge branch 'develop' of github.com:holochain/holochain into 2021-09-21-post-commit by @thedavidmeister
- Merge pull request #984 from holochain/2021-09-06-schedule-do-something by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2021-09-06-schedule-do-something by @thedavidmeister
- Merge pull request #998 from holochain/release-20210922.083906 by @steveej
- Create a release from branch release-20210922.083906
- Merge pull request #997 from holochain/pr/fixup-documentation-urls by @steveej
- Merge pull request #978 from holochain/more-gossip-fixes-4 by @freesig
- Merge branch 'develop' into more-gossip-fixes-4
- Merge branch 'develop' into more-gossip-fixes-4 by @freesig
- Merge branch 'develop' of github.com:/holochain/holochain into more-gossip-fixes-4 by @freesig
- Merge branch 'develop' of github.com:holochain/holochain into 2021-09-06-schedule-do-something by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2020-09-06-schedule-do-something by @thedavidmeister
- Update changelog by @thedavidmeister
- Infallible externs by @thedavidmeister
- Wip on post commit callback by @thedavidmeister
- Merge pull request #983 from holochain/release-20210916.085414 by @steveej
- Create a release from branch release-20210916.085414
- Merge pull request #981 from holochain/holo-hash-no-default-features by @steveej
- HDK builds by @maackle
- Set default features to something sensible by @maackle
- CHANGELOG by @maackle
- Adjust feature usage in holo_hash so it builds under --no-default-features by @maackle
- Merge pull request #962 from holochain/timestamp-millis by @maackle
- Merge pull request #970 from holochain/timestamp-new-crate by @maackle
- Merge pull request #971 from holochain/timestamp-standardization by @maackle
- Merge pull request #972 from holochain/release-20210901.105419 by @steveej
- Applying develop versions to unpublished crates
- Merge remote-tracking branch 'upstream/develop' into release-20210901.105419 by @steveej
- Merge pull request #967 from holochain/bump-ghost-actor by @neonphog
- Work around the dry-run release automation... by @neonphog
- Create a release from branch release-20210901.105419
- Merge pull request #948 from holochain/release-20210825.101130 by @steveej
- Create a release from branch release-20210825.101130
- Merge pull request #931 from holochain/release-20210817.185301 by @steveej
- Create a release from branch release-20210817.185301
- Merge pull request #881 from holochain/pr/test-parallel-install by @steveej
- Further debug, test and bench concurrent app installs by @steveej
- Merge pull request #770 from holochain/handy-holohash-ext by @maackle
- Merge remote-tracking branch 'origin/develop' into handy-holohash-ext by @maackle
- Merge pull request #916 from holochain/2021-07-07-countersigning-integrity-check by @thedavidmeister
- Merge pull request #895 from holochain/2021-07-28-accept-preflight by @thedavidmeister
- Merge branch '2021-07-07-countersigning-integrity-check' into 2021-07-28-accept-preflight by @thedavidmeister
- Merge pull request #781 from holochain/hdk-uses-featureless-zome-types by @maackle
- Merge remote-tracking branch 'origin/develop' into hdk-uses-featureless-zome-types by @maackle
- Merge remote-tracking branch 'origin/develop' into hdk-uses-featureless-zome-types by @maackle
- Properly use fixturator feature in test wasms by @maackle
- Use no default-features for zome_types in hdk by @maackle
- Able to create and update countersigned entries by @thedavidmeister
- Lint by @thedavidmeister
- Compiling chain lock by @thedavidmeister
- Merge branch 'develop' into 2021-07-28-accept-preflight by @thedavidmeister
- Merge pull request #893 from holochain/release-20210722.172107 by @steveej
- Fix unused imports by @steveej
- Create a release from branch release-20210722.172107
- Wip on locking preflights by @thedavidmeister
- Merge remote-tracking branch 'origin/develop' into handy-holohash-ext by @maackle
- Merge pull request #885 from holochain/experiment/release-debug by @steveej
- Applying develop versions to changed crates
  - The following crates changed since their most recent release and are therefore increased to a develop version:
  - Holochain-0.0.102-dev.0 - holochain_wasm_test_utils-0.0.2-dev.0 - holochain_cli-0.0.3-dev.0 - holochain_util-0.0.2-dev.0 - mr_bundle-0.0.2-dev.0 - holochain_zome_types-0.0.4-dev.0 - holochain_test_wasm_common-0.0.2-dev.0 - hdk_derive-0.0.4-dev.0 - fixt-0.0.4-dev.0 - holochain_keystore-0.0.2-dev.0 - hdk-0.0.102-dev.0 - holochain_sqlite-0.0.2-dev.0 - holochain_cli_bundle-0.0.2-dev.0 - holo_hash-0.0.4-dev.0 - holochain_state-0.0.2-dev.0 - holochain_p2p-0.0.2-dev.0 - holochain_cli_sandbox-0.0.3-dev.0 - kitsune_p2p-0.0.2-dev.0 - holochain_websocket-0.0.2-dev.0 - holochain_types-0.0.2-dev.0 - holochain_cascade-0.0.2-dev.0 - holochain_conductor_api-0.0.2-dev.0 - kitsune_p2p_types-0.0.2-dev.0 - kitsune_p2p_proxy-0.0.2-dev.0 - kitsune_p2p_transport_quic-0.0.2-dev.0
- Revert "setting develop versions to conclude 'release-20210624.155736'" by @steveej
- Merge pull request #877 from holochain/facts by @maackle
- Use new holochain_serialized_bytes release by @maackle
- Clippy by @maackle
- Merge remote-tracking branch 'origin/develop' into facts by @maackle
- Merge pull request #842 from holochain/2021-06-16-countersigning by @thedavidmeister
- Merge branch 'develop' into 2021-06-16-countersigning by @thedavidmeister
- Merge pull request #812 from holochain/2021-05-26-must by @thedavidmeister
- Merge remote-tracking branch 'origin/develop' into 2021-05-26-must by @thedavidmeister
- Base data structures for countersigning by @thedavidmeister
- Merge branch 'develop' into 2021-06-16-countersigning by @thedavidmeister
- Merge pull request #865 from holochain/release-20210624.155736 by @steveej
- Merge remote-tracking branch 'upstream/develop' into release-20210624.155736 by @steveej
- Merge pull request #862 from holochain/pr/tokio-helper-fix-timeout by @steveej
- Fix tokio_helper bug where timeout relied on an existing reactor by @steveej
- Setting develop versions to conclude 'release-20210624.155736'
- Release-20210624.155736
- Wip on countersigning by @thedavidmeister
- Impl Arbitrary for Element by @maackle
- Merge remote-tracking branch 'origin/develop' into dhtop-arbitrary by @maackle
- Properly implement Arbitrary for HoloHash by @maackle
- Remove circular dep in features by @maackle
- Merge remote-tracking branch 'origin/develop' into handy-holohash-ext by @maackle
- Merge remote-tracking branch 'origin/develop' into sqlite by @maackle
- Merge branch 'develop' of github.com:holochain/holochain into sqlite by @freesig
- Merge pull request #767 from holochain/sqlite-integrating by @freesig
- Merge branch 'develop' of github.com:holochain/holochain into sqlite by @freesig
- Merge branch 'develop' of github.com:holochain/holochain into sqlite by @freesig
- Spike on publish and sys validation by @freesig
- A test passes for agent activity query by @maackle
- Make scratch not generic and clean up traits by @freesig
- Merge remote-tracking branch 'origin/develop' into sqlite by @maackle
- Merge pull request #707 from holochain/decomplect-keystore-and-db by @maackle
- Remove dependency on holochain_keystore by holochain_sqlite by @maackle
- Merge remote-tracking branch 'origin/develop' into sqlite by @maackle
- Merge remote-tracking branch 'origin/develop' into sqlite by @maackle
- Completely remove rkv by @maackle
- Merge remote-tracking branch 'origin/develop' into handy-holohash-ext by @maackle
- Merge pull request #755 from holochain/experiment/macos-unstable-nixpkgs by @thedavidmeister
- Clips by @thedavidmeister
- Merge pull request #723 from holochain/pr/release-automation-milestone-1 by @steveej
- Add missing changelogs mostly with `unreleasable: true`, adjust version numbers by @steveej
  - The version change is necessary to adhere to our defined restriction of only wanting to release 0.0.X versions until we're ready for more stability promising version numbers.
- Merge pull request #779 from holochain/pr/holochain_util_and_apache_licenses by @steveej
- Merge ffs and tokio_helper into a new holochain_util crate by @steveej
- Change some licenses to Apache-2.0 by @steveej
- Re-add test_utils feature to holo_hash by @maackle
- Merge remote-tracking branch 'origin/develop' into handy-holohash-ext by @maackle
- Merge pull request #708 from holochain/2021-03-22-deepkey-tweaks by @thedavidmeister
- Merge branch 'develop' into 2021-03-22-deepkey-tweaks by @thedavidmeister
- Unpub fields by @thedavidmeister
- Merge branch '2021-03-22-deepkey-tweaks' of github.com:holochain/holochain into 2021-03-22-deepkey-tweaks by @thedavidmeister
- Merge branch 'develop' into 2021-03-22-deepkey-tweaks by @thedavidmeister
- Lint by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2021-03-22-deepkey-tweaks by @thedavidmeister
- Wip on deepkey tweaks by @thedavidmeister
- Add .to_hash() and .into_hashed() for any HashableContent by @maackle
- Merge pull request #753 from holochain/pr/restructure-changelogs by @steveej
- Manually prepare all CHANGELOG files for automation by @steveej
- Merge pull request #731 from holochain/tx2-kitsune by @neonphog
- Merge pull request #735 from holochain/tx2-errors by @neonphog
- Additional tracing by @neonphog
- Merge pull request #726 from holochain/happ-uuid by @maackle
- Allow changing all UUIDs en masse when installing an AppBundle by @maackle
- Merge pull request #689 from holochain/2021-03-11-ephemeral-sign by @thedavidmeister
- Merge branch 'develop' into 2021-03-11-ephemeral-sign by @thedavidmeister
- Merge pull request #679 from holochain/pr/tokio-1.2 by @steveej
- Bump tokio to v1 and introduce tokio_helper by @steveej
- Merge branch '2020-03-11-ephemeral-sign' of github.com:holochain/holochain into 2021-03-11-ephemeral-sign by @thedavidmeister
- Merge branch 'develop' into 2021-03-11-ephemeral-sign by @thedavidmeister
- Merge pull request #674 from holochain/2021-03-03-mock-hdk by @thedavidmeister
- Move hosthdk init into hdk and out of extern by @thedavidmeister
- Conductor sign ephemeral by @thedavidmeister
- Hdk ephemeral sign by @thedavidmeister
- Lint by @thedavidmeister
- Wip on testing mock by @thedavidmeister
- Wip on wasm unit tests by @thedavidmeister
- Merge pull request #669 from holochain/release-20210226.155101 by @steveej
- Specify holochain_zome_types version by @steveej
- Merge pull request #667 from holochain/release-20210226.155101 by @steveej
- Merge branch 'main' into release-20210226.155101 by @steveej
- Merge branch 'develop' of https://github.com/holochain/holochain into main
- Merge pull request #355 from holochain/develop by @zippy
- Multi-crate release performed on 2021-02-26 by @steveej
- Merge pull request #662 from holochain/pr/prepare-release-hdk-0.0.100-alpha.0 by @steveej
- Make it build with default-features by @steveej
  - Fixing the standalone build for `holo_hash` as it's about to be published in the process of publishing the `hdk`.
- Merge pull request #664 from holochain/revert-kitsune-license by @steveej
- Merge pull request #661 from holochain/pr/cal-license-everywhere by @steveej
- Use CAL-1.0 license file in all crates by @thedavidmeister
- Merge pull request #651 from holochain/happ-bundles-3 by @maackle
- Unpatch wasmer and serialization by @thedavidmeister
- Correct dep versions, add binary integration roundtrip test by @maackle
- Update copyright year by @maackle
- Merge remote-tracking branch 'origin/develop' into happ-bundles-3 by @maackle
- Docs (#639) by @thedavidmeister
- Clarify docs for HoloHashB64 by @maackle
- Attempt to unify deserialization for HoloHash and HoloHashB64 by @maackle
- Merge branch 'happ-bundles-1-point-5' into happ-bundles-2 by @maackle
- Merge branch 'mr-bundle' into happ-bundles-1-point-5 by @maackle
- Merge branch 'happ-bundles-1' into mr-bundle by @maackle
- Merge branch 'yaml-dna' into happ-bundles-1 by @maackle
- Merge branch 'develop' into yaml-dna by @freesig
- Actually make a cloned DNA. Make DNA hashing sync. by @maackle
- Merge branch 'happ-bundles-cli' into happ-bundles-2 by @maackle
- Merge remote-tracking branch 'origin/develop' into happ-bundles-1-point-5 by @maackle
- Merge remote-tracking branch 'origin/develop' into happ-bundles-2 by @maackle
- Merge pull request #635 from holochain/feature-timestamp by @neonphog
- Fmt+clippy by @neonphog
- Improve compatibility of holochain types with Zome WASM by @pjkundert
- Merge pull request #624 from holochain/2021-02-02-tracing by @thedavidmeister
- Wip on tracing from wasm by @thedavidmeister
- Basic idea of cloning cells, with lots of TODOs by @maackle
- Merge branch 'mr-bundle' into happ-bundles-2 by @maackle
- Merge branch 'happ-bundles-1' into mr-bundle by @maackle
- Merge branch 'happ-bundles-1' into happ-bundles-2 by @maackle
- Merge remote-tracking branch 'origin/develop' into happ-bundles-1 by @maackle
- Clippy by @maackle
- Merge remote-tracking branch 'origin/happ-bundles' into bundle-refactor by @maackle
- Base64-encodable HoloHash, and properly serializing AppManifestV1 by @maackle
- Use Yaml for DNA manifest instead of JSON by @maackle
- Merge remote-tracking branch 'origin/develop' into mr-bundle by @maackle
- Merge pull request #576 from holochain/2021-01-06-into-inner by @thedavidmeister
- Wip on new wasmer by @thedavidmeister
- Relax serde deps by @maackle
- Merge pull request #530 from holochain/crate-reorg by @maackle
- Cargo update by @maackle
- Bump serialization and wasmer by @maackle
- Merge remote-tracking branch 'origin/develop' into crate-reorg by @maackle
- Merge pull request #559 from holochain/2020-12-15-crypto-box by @thedavidmeister
- Lint tests by @thedavidmeister
- Merge by @thedavidmeister
- Merge branch '2020-12-01-secretbox' of github.com:holochain/holochain into 2020-12-15-crypto-box by @thedavidmeister
- Wip on secretbox by @thedavidmeister
- Crypto box wip by @thedavidmeister
- Merge remote-tracking branch 'origin/develop' into crate-reorg by @maackle
- Merge pull request #546 from holochain/remote_signal by @freesig
- Bump versions by @freesig
- Merge remote-tracking branch 'origin/develop' into crate-reorg by @maackle
- Merge pull request #551 from holochain/pr/nix-bump-rust-1.48.0 by @steveej
- Respect clippy lints for rust 1.48.0 by @steveej
- Fmt by @maackle
- Fix up unused deps and feature flag alignment for zome_types by @maackle
- Hdk3 + hdk3_derive check out by @maackle
- Move zome_types to own crate, but with fixturator errors by @maackle
- Monolith checks out by @maackle
- Add some necessary deps to monolith Cargo.toml by @maackle
- Revert "Remove all Cargo.toml from moved crates" by @maackle
- Move wasm_workspace; all lib.rs become mod.rs by @maackle
- Remove all Cargo.toml from moved crates by @maackle
- Cargo fmt -- --config=flatten_imports=true (using rustfmt fork) by @maackle
- Merge pull request #512 from holochain/inline-zome by @maackle
- Cargo fmt -- --config=merge_imports=true (unstable rustfmt) by @maackle
- Merge branch 'host-fn-api-struct' into inline-zome by @maackle
- Merge pull request #489 from holochain/2020-11-16-export-hdk-externs by @thedavidmeister
- Remove old docs by @thedavidmeister
- Mod based extern mapping by @thedavidmeister
- Remove old docs by @thedavidmeister
- Mod based extern mapping by @thedavidmeister
- Fmt by @maackle
- Make ZomeCallInvocation take Zome instead of ZomeName by @maackle
- Inline zome enum hooked up by @maackle
- Fix warnings by @maackle
- Refactor DnaFile to use DnaDefHashed by @maackle
- Merge pull request #468 from holochain/2020-11-07-now by @thedavidmeister
- Merge pull request #471 from holochain/holohash-39 by @maackle
- Fix test by @maackle
- Use const by @maackle
- Adds a test that ensures that HoloHash serializes to a raw byte array by @maackle
- Merge pull request #466 from holochain/2020-11-04-bootstrap by @thedavidmeister
- Merge branch 'develop' of github.com:holochain/holochain into 2020-11-04-bootstrap by @thedavidmeister
- Merge pull request #459 from holochain/holohash-39 by @maackle
- Apply suggestions from code review by @maackle
- Remove Default impls by @maackle
- WIP by @maackle
- Clean up diagram by @maackle
- Use debug asserts for assert_length by @maackle
- Add doc diagram. Full -> Untyped, Raw -> Full by @maackle
- With_pre_hashed -> from_raw_32 by @maackle
- More usage of consts by @maackle
- Fix prefix bug by @maackle
- Retype agent hash as entry hash; fix KeystoreSender slice problem by @maackle
- Big rename, hunting down remaining failure causes by @maackle
- Fix serialization problem. Test failures remain. by @maackle
- Using 39 bytes where appropriate by @maackle
- Use proper deserialization errors by @maackle
- Use a raw byte vec for HoloHash serialization by @maackle
- Take 1: intermediary type for holohash de/serialization by @maackle
- Directly test bootstrap by @thedavidmeister
- Merge pull request #441 from holochain/fix_fixt by @maackle
- Merge branch 'develop' into fix_fixt by @freesig
- Merge pull request #408 from holochain/cache_val_pack by @freesig
- Merge branch 'develop' into cache_val_pack by @freesig
- Annotate all ignored tests by @maackle
- Remote drops by @freesig
- Merge branch 'get_agent_activity_cascade' of https://github.com/holochain/holochain into cache_val_pack by @freesig
- Merge pull request #414 from holochain/custom_val_pack by @freesig
- Merge branch 'cache_val_pack' of https://github.com/holochain/holochain into custom_val_pack by @freesig
- Custom val package by @freesig
- Merge pull request #417 from holochain/fixturator-seed by @maackle
- Make singleton rng private by @maackle
- Cleanup by @maackle
- Clip by @maackle
- Prevent deadlocks by getting a lock at every rng mutation by @maackle
- Move rng into own module by @maackle
- Merge remote-tracking branch 'origin/develop' into fixturator-seed by @maackle
- Merge pull request #410 from holochain/2020-10-05-kitsune-store-event by @neonphog
- Pull holo_hash out of kitsune by @thedavidmeister
- Use internal fixt::rng everywhere by @maackle
- Use Arc<Mutex<StdRng>>, convert some fixturators by @maackle
- Add lazy_static FIXTURATOR_RNG by @maackle
- Merge pull request #399 from holochain/get_validation_package by @freesig
- Merge branch 'get_validation_package' of https://github.com/holochain/holochain into get_validation_package by @freesig
- Merge pull request #344 from holochain/val_package by @freesig
- Name change by @freesig
- Merge branch 'validate_entry_id' of https://github.com/holochain/holochain into val_package by @freesig
- Merge pull request #385 from holochain/chain_rollback by @freesig
- Add consts by @freesig
- Change to chain to sub chain by @freesig
- Merge branch 'validate_entry_id' of https://github.com/holochain/holochain into val_package by @freesig
- Merge pull request #367 from holochain/2020-09-18-sign by @thedavidmeister
- Working sign in wasm by @thedavidmeister
- Merge branch 'app_val' of https://github.com/holochain/holochain into val_package by @freesig
- Merge pull request #319 from holochain/lair-mock-keystore by @neonphog
- Merge branch 'list-cells' into lair-mock-keystore by @neonphog
- Merge pull request #343 from holochain/2020-09-13-hdk-sweep by @thedavidmeister
- Merge by @thedavidmeister
- Merge branch 'develop' into rsm-release by @zippy
- Update urls by @zippy
- Merge branch 'develop' into 2020-09-13-hdk-sweep by @thedavidmeister
- Merge branch '2020-08-25-hdk-docs' of github.com:Holo-Host/holochain into 2020-09-13-hdk-sweep by @thedavidmeister
- Wip on hdk docs by @thedavidmeister
- Merge branch 'develop' into lair-mock-keystore by @neonphog
- Merge pull request #338 from Holo-Host/guillemcordoba-patch-1 by @guillemcordoba
- Small fix in holo_hash documentation by @guillemcordoba
- Merge branch 'develop' into lair-mock-keystore by @neonphog
- Use lair test keystore by @neonphog
- First pass at validation package by @freesig
- Merge pull request #337 from Holo-Host/2020-09-10-holonix-stable by @maackle
- Merge branch 'develop' of github.com:Holo-Host/holochain into 2020-09-10-holonix-stable by @thedavidmeister
- Merge pull request #335 from Holo-Host/query-chain by @maackle
- Merge branch 'develop' into query-chain by @maackle
- Merge pull request #325 from Holo-Host/remove-async-hash-types by @maackle
- Merge remote-tracking branch 'origin/develop' into remove-async-hash-types by @maackle
- Remove temporary HashTypeAsync from 3 sync hash types by @maackle
- Merge remote-tracking branch 'origin/develop' into query-chain by @maackle
- Test for EntryType filter, and much Fixturator refactoring to enable it by @maackle
- Stable rust from holonix by @thedavidmeister
- Merge pull request #317 from Holo-Host/2020-08-30-cap-claim by @thedavidmeister
- Merge branch 'develop' of github.com:Holo-Host/holochain into 2020-08-30-cap-claim by @thedavidmeister
- Merge pull request #294 from Holo-Host/sys_val by @freesig
- Merge branch 'develop' of https://github.com/Holo-Host/holochain into sys_val by @freesig
- Merge pull request #321 from Holo-Host/integrate_pending by @freesig
- Dependancies working by @freesig
- All prefixes done by @freesig
- Merge pull request #309 from Holo-Host/sys_val_reader by @freesig
- Merge pull request #315 from Holo-Host/sync-hashing by @maackle
- Merge branch 'entry-size-limit' into sync-hashing by @maackle
- Merge pull request #313 from Holo-Host/2020-08-28-cap-grant by @thedavidmeister
- Merge branch 'develop' into 2020-08-28-cap-grant by @thedavidmeister
- Can commit cap grants by @thedavidmeister
- Split Cas into sync and async versions by @maackle
- Convert some things to sync hashing by @maackle
- Use traits to specify "syncness" of hash types by @maackle
- Remove from_data by @maackle
- Merge pull request #295 from Holo-Host/reading-rainbow by @maackle
- Update to serialized_bytes 0.0.43 by @maackle
- Apply suggestions from code review by @maackle
- Cleanup by @maackle
- Merge branch 'develop' of https://github.com/Holo-Host/holochain into reading-rainbow by @freesig
- Merge pull request #298 from Holo-Host/2020-08-20-hdk-sweep by @thedavidmeister
- Lint by @thedavidmeister
- Merge branch 'develop' of github.com:Holo-Host/holochain into 2020-08-20-hdk-sweep by @thedavidmeister
- Merge pull request #296 from Holo-Host/holohash-primitive-entry-hashes by @maackle
- Update crates/holo_hash/src/aliases.rs by @maackle
- Wip on hdk_entry macro by @thedavidmeister
- Implement as-at check and add terrible HACK for AnyDhtHash keys by @maackle
- Make BufKey an implementable trait by @maackle
- Accept 32 or 36 bytes by @maackle
- Make core/full bytes for holohash consistent and clear by @maackle
- Uncomment iter code, untested by @maackle
- Add From<Vec<u8>> for primitive HoloHash by @maackle
- Fix test by @maackle
- Make HoloHash::retype crate-private by @maackle
- Make Entry a primitive hash type by @maackle
- Merge pull request #293 from Holo-Host/2020-08-14-hdk-review by @thedavidmeister
- Wip on hdk review by @thedavidmeister
- Merge pull request #279 from Holo-Host/demo_freesig by @freesig
- To_vec_named -> encode by @freesig
- Merge pull request #275 from Holo-Host/hashtype-serde-weirdness by @thedavidmeister
- Bump crates by @thedavidmeister
- Merge branch 'hashtype-serde-weirdness' of github.com:Holo-Host/holochain into hashtype-serde-weirdness by @thedavidmeister
- Update to new published dep versions by @maackle
- Revert "Add wasmer_guest patch everywhere" by @maackle
- Add wasmer_guest patch everywhere by @maackle
- Use sb's string variant encoding, add json friendly deserialize for hashtype by @maackle
- Change order of tests by @maackle
- Change order of tests to show inconsistency by @maackle
- WIP test to show holo_hash serde weirdness by @maackle
- Merge pull request #269 from Holo-Host/integration-queue-refactor by @maackle
- Merge remote-tracking branch 'origin/develop' into integration-queue-refactor by @maackle
- Merge pull request #268 from Holo-Host/holohash-serialization-disambiguation by @maackle
- Add README by @maackle
- Disambiguate composite serialization with from/into intermediate form by @maackle
- Add failing tests; organization by @maackle
- Fix test by @maackle
- Merge pull request #263 from Holo-Host/update_hostfn by @freesig
- Change host fn names by @freesig
- Merge pull request #248 from Holo-Host/housekeeping by @maackle
- Remove deprecated function by @maackle
- :with_data -> from_content; panics instead of erroring by @maackle
- HeaderAddress -> HeaderHash by @maackle
- Merge pull request #241 from Holo-Host/holohash-refactor by @maackle
- Merge remote-tracking branch 'origin/develop' into holohash-refactor by @maackle
- Merge pull request #245 from Holo-Host/move_headers by @thedavidmeister
- All headers moved by @freesig
- Remove unneeded file by @maackle
- Add serialized-bytes feature flag by @maackle
- Collapse all holo_hash into single crate with feature flags by @maackle
- WIP by @maackle
- Holo_hash -> holo_hash_ext; holo_hash_core -> holo_hash by @maackle
- Merge pull request #238 from Holo-Host/holohash-refactor by @thedavidmeister
- Lint by @thedavidmeister
- Remove futures from holo hash deps by @thedavidmeister
- Clean up a few deprecations by @maackle
- HoloHashImpl -> HoloHash by @maackle
- HoloHash -> HoloHashOf by @maackle
- Flesh out holo_hash docs by @maackle
- Fix fixturator by @maackle
- Fix slow_tests by @maackle
- Merge remote-tracking branch 'origin/develop' into holohash-refactor by @maackle
- Cleanup, comments, clippy by @maackle
- Fix warnings by @maackle
- HashableContent can be prehashed by @maackle
- Simplified HashableContent bound by @maackle
- Need to special-case HashableContent for Agent within Entry by @maackle
- Adds dubiously useful hash_type_enum by @maackle
- Whoops, actually add the holo_hash crates! by @maackle
- Need to refactor SignedHeaderHashed now, so it fits into the CasBuf by @maackle
- Uh oh! Orphan rule! by @maackle
- Merge pull request #225 from Holo-Host/rm-2020 by @neonphog
- Rm 2020 by @neonphog
- Merge pull request #205 from Holo-Host/cargo-udeps by @timotree3
- Remove unused dependencies by @timotree3
- Merge pull request #177 from Holo-Host/2020-06-04-call-remote-db by @thedavidmeister
- Merge branch 'develop' of github.com:Holo-Host/holochain-2020 into 2020-06-04-call-remote-db by @thedavidmeister
- Merge pull request #189 from Holo-Host/2020-06-12-wasm-perf by @thedavidmeister
- Merge branch 'develop' into 2020-06-12-wasm-perf by @thedavidmeister
- Wasm security fixes by @thedavidmeister
- Wip on wasm perf by @thedavidmeister
- Wip on removing send by @thedavidmeister
- Merge pull request #186 from Holo-Host/docs by @maackle
- Merge remote-tracking branch 'origin/develop' into docs by @maackle
- Merge pull request #188 from Holo-Host/readmes by @timotree3
- Holochain lib readme plus cargo toml fixes by @zippy
- Gen README for holo_hash by @zippy
- Merge pull request #185 from Holo-Host/v-receipt-db by @neonphog
- Merge branch 'develop' into v-receipt-db by @neonphog
- Merge pull request #161 from Holo-Host/cascade-meta by @freesig
- Merge branch 'develop' of https://github.com/Holo-Host/holochain-2020 into cascade-meta by @freesig
- Pr fixes by @freesig
- Merge branch 'develop' of https://github.com/Holo-Host/holochain-2020 into cascade-meta by @freesig
- Merge remote-tracking branch 'origin/develop' into cascade-meta by @maackle
- Merge branch 'links_meta' of https://github.com/Holo-Host/holochain-2020 into cascade-meta by @freesig
- WIP, failing partially implemented test of update by @maackle
- Working validation receipts db by @neonphog
- Merge pull request #184 from Holo-Host/fixturator-clippy-fixes by @neonphog
- Docs by @maackle
- Merge pull request #168 from Holo-Host/fix-hash-blocking by @neonphog
- Merge pull request #162 from Holo-Host/cas-integrity by @maackle
- Apply suggestions from code review by @maackle
- Use MustBoxFuture in holo_hash and other places by @maackle
- Add Sized constraint by @maackle
- Apply suggestions from code review by @maackle
- Some renames by @maackle
- Massive refactor of CasBuf to include integrity check by @maackle
- Implement Hashable trait by @maackle
- Fix doc tests by @maackle
- Add HoloHashHash trait by @maackle
- Merge pull request #158 from Holo-Host/sorted-deps by @neonphog
- Merge branch 'develop' into sorted-deps by @neonphog
- Merge pull request #153 from Holo-Host/2020-05-21-ribosome-tests by @thedavidmeister
- Merge branch 'develop' of github.com:Holo-Host/holochain-2020 into 2020-05-21-ribosome-tests by @thedavidmeister
- More tests for fixts by @thedavidmeister
- Wip on enum fixturator by @thedavidmeister
- Lint by @thedavidmeister
- Less boilerplate in fixturators by @thedavidmeister
- Wip on enum fixturator by @thedavidmeister
- Merge branch '2020-05-21-ribosome-tests' of github.com:Holo-Host/holochain-2020 into 2020-05-21-ribosome-tests by @thedavidmeister
- Mocked tests for call iterator by @thedavidmeister
- Sort Cargo.toml deps using cargo-sort-ck by @neonphog
- Merge pull request #157 from Holo-Host/new-ghost-api by @neonphog
- Pin paste by @neonphog
- Checkpoint by @neonphog
- Merge pull request #150 from Holo-Host/address-refactor by @maackle
- Re-insitute the renamings by @maackle
- Merge remote-tracking branch 'origin/develop' into address-refactor by @maackle
- Merge pull request #108 from Holo-Host/2020-04-30-wasm-callbacks by @thedavidmeister
- Wrapping dna hashes by @thedavidmeister
- Merge branch 'develop' of github.com:Holo-Host/holochain-2020 into 2020-04-30-wasm-callbacks by @thedavidmeister
- Constants for critical perf paths by @thedavidmeister
- Merge branch 'develop' of github.com:Holo-Host/holochain-2020 into 2020-04-30-wasm-callbacks by @thedavidmeister
- Init tests wip by @thedavidmeister
- Wip on validaton callback by @thedavidmeister
- Temporary revert naming to make merge easier by @maackle
- EntryHash -> EntryContentHash by @maackle
- Merge pull request #128 from Holo-Host/make-hashed by @neonphog
- Use verbose macro by @neonphog
- Merge branch 'develop' into make-hashed by @neonphog
- Merge pull request #125 from Holo-Host/2020-05-12-bump-wasmer by @neonphog
- Bump wasmer by @thedavidmeister
- Fmt by @neonphog
- Apply suggestions from code review by @neonphog
- Minor streamline of Hashed api by @neonphog
- Checkpoint by @neonphog
- Checkpoint by @neonphog
- Merge pull request #94 from Holo-Host/combined-headers-and-entries by @zippy
- Spurious executable fix by @zippy
- Rename AgentHash -> AgentPubKey by @zippy
- Merge pull request #92 from Holo-Host/2020-04-22-fixtures by @zippy
- Clippy by @thedavidmeister
- Macro based fixturator by @thedavidmeister
- WIP on fixturator by @thedavidmeister
- Basic_test for all primitive fixtures by @thedavidmeister
- Wip on fixturator by @thedavidmeister
- Random size fixt by @thedavidmeister
- New type fixt macro by @thedavidmeister
- String fixtures by @thedavidmeister
- Chars for fixtures by @thedavidmeister
- Float fixtures by @thedavidmeister
- Merge branch '2020-04-22-fixtures' of github.com:Holo-Host/holochain-2020 into 2020-04-22-fixtures by @thedavidmeister
- Merge branch 'develop' into 2020-04-22-fixtures by @thedavidmeister
- Merge pull request #91 from Holo-Host/2020-04-16-wasm-db by @zippy
- Wasm db i think by @thedavidmeister
- Add int fixtures by @thedavidmeister
- Wip on fixtures by @thedavidmeister
- Merge pull request #81 from Holo-Host/2020-04-16-hash-cas by @thedavidmeister
- Update crates/holo_hash/src/lib.rs by @thedavidmeister
- Merge branch 'develop' of github.com:Holo-Host/holochain-2020 into 2020-04-16-hash-cas by @thedavidmeister
- Merge pull request #56 from Holo-Host/TK-01224 by @maackle
- Bump serialized-bytes dep by @maackle
- WIP by @maackle
- Clippy works by @thedavidmeister
- Fmt by @thedavidmeister
- Wip on using holo hash by @thedavidmeister
- Merge pull request #67 from Holo-Host/apply-clippy-fixes by @timotree3
- Merge branch 'develop' into apply-clippy-fixes by @zippy
- Merge pull request #69 from Holo-Host/holo_hash_followon by @neonphog
- Address some after-the-fact code review by @neonphog
- Apply clippy fixes by @timotree3
- Integrate holo_hash_core into holo_hash by @neonphog
- Initial working holo-hash : ) by @neonphog
- Initial commit by @holochain-release-automation2

### First-time Contributors

- @ made their first contribution in [#3](https://github.com/holochain/holochain-hdi/pull/3)
- @veeso made their first contribution
- @ThetaSinner made their first contribution
- @holochain-release-automation2 made their first contribution
- @jost-s made their first contribution
- @mattyg made their first contribution
- @grandima made their first contribution
- @matthme made their first contribution
- @pdaoust made their first contribution
- @neonphog made their first contribution
- @ddd-mtl made their first contribution
- @cdunster made their first contribution
- @shandongzhejiang made their first contribution
- @highcloudwind made their first contribution
- @c12i made their first contribution
- @nphias made their first contribution
- @guqicun made their first contribution
- @maackle made their first contribution
- @xiaoxianBoy made their first contribution
- @steveej made their first contribution
- @joshuavial made their first contribution
- @thedavidmeister made their first contribution
- @Seb33300 made their first contribution
- @zippy made their first contribution
- @github-actions[bot] made their first contribution
- @robbiecarlton made their first contribution
- @abe-njama made their first contribution
- @freesig made their first contribution
- @timotree3 made their first contribution
- @pjkundert made their first contribution
- @guillemcordoba made their first contribution

Changes made before these crates moved out of
[holochain/holochain](https://github.com/holochain/holochain) are listed in
that repository's `CHANGELOG.md` up to the split.
