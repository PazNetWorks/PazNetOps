# PazNetOps

For repeatable customer onboarding, use the [standard site deployment workflow](docs/REPEATABLE_SITE_DEPLOYMENT.md). The generic signed setup combines install/resume, private enrollment/credential input, updater installation and readiness reporting. Its new native release validation and organization signing remain pending.

A network inventory and telemetry service for dedicated Windows 10/11 collectors at customer sites. Collectors use outbound HTTPS for enrollment/control/uploads, approved UniFi API reads, SNMP, and bounded reachability checks. Optional IPFIX/NetFlow and resolver/Zeek JSON metadata inputs require explicit signed policy and a real observation source.

The implementation is in progress against the full [build plan](outputs/NETWORK_ANALYZER_BUILD_PLAN.md). [Implementation status](docs/implementation-status.json) records local evidence and remaining work separately from acceptance. Real isolated PostgreSQL17.11 and ClickHouse26.8.15.10, actual local mutual TLS, encrypted archives, and logical database restoration have been exercised. Actual Windows service/hardware, current UniFi models, hosted signed-in Supabase acceptance, publisher signing, and field/load/disaster-recovery acceptance are separate gates. A successful build is not production acceptance.

The current [local verification record](outputs/verification-endpoint-cache-final.json) and [readable handoff](docs/LOCAL_VERIFICATION.md) cover fresh 19-migration database race/restore checks, the restricted API image, matching normal/race telemetry smokes and the bounded lookup replacement. The 20k/200k/1m read stages each passed 480 queries and six semantic gates; worst p99 at one million was 31.5 ms. A populated-history recovery run completed one million cache rows, preserved original evidence and recovered a timed-out coverage proof through a bounded retry. The older generated-column migration remains rejected for online populated history. These are local synthetic results; Windows service/hardware, hosted signed-in acceptance, model support and sustained capacity remain separate gates. The Windows review package is unsigned and the protected installer rejects it.


The initial [hosted staging foundation](outputs/hosted-staging-setup.json) is now deployed at [paznetops-staging.vercel.app](https://paznetops-staging.vercel.app), with NetOps Supabase, a separate API and native TLS collector ingress, and a private health worker. All20 migrations, native restricted-role/TLS startup, schema privileges, readiness, origin restrictions and certificate denial were checked. MRC runs the locally signed pilot version13 and has passed actual HTTPS updates. The [October5 health audit](outputs/windows-health-audit-20261005.json) records the native and hosted checks and a repaired scheduler retention bug. The first administrator account, hosted enrollment and real signed-in customer/UniFi acceptance remain outstanding. Metadata collection stays disabled pending actual source and capacity acceptance.

The [Windows lab kit](dist/windows-lab/paznetops-windows-lab-0.1.0-dev.10.zip) runs without development tools or service installation. The [user-supplied October 5 result](outputs/windows-lab-0.1.0-dev.10-results-20261005.json) has been reviewed: all four checks pass on native x64, reported build26200.8875/25H2/Core, with Ethernet index11 up and index12 down. These unsigned component checks precede the separate signed service installation and update checks. See [Windows operations](scripts/windows/README.md) for the current pilot and remaining field acceptance.
The corrected [MRC pilot installer](dist/windows-pilot/paznetops-mrc-install-0.1.0-dev.11-r3.zip) contains collector11, update12 and the signed fleet-update task. MRC was installed through Assist and successfully updated from11 to12 through the hosted HTTPS feed under SYSTEM. Configuration, encrypted credentials and the spool key survived the service restart. Production signing, hosted enrollment, UniFi and reboot/failure acceptance remain open. See [installation and fleet update procedures](docs/COLLECTOR_UPDATES.md) and the [native update report](outputs/mrc-native-update-result-20261005.json).

## Source layout

| Location | Purpose |
|---|---|
| `cmd/collector`, `internal/collector` | Windows SCM service, protected local identity/credentials, signed policy, scheduling, encrypted spool and uploader |
| `internal/adapters` | Source-bound network scope, official UniFi reads, bounded SNMP and native Windows IP Helper/ICMP |
| `internal/topology` | Bounded typed port/alias resolver with explicit provenance, ambiguity and reciprocal evidence; automatic source association remains a hardware gate |
| `internal/location` | Bounded discrete MAC/IP learning evidence with same-acquisition mapping proofs and explicit physical/uplink uncertainty |
| `internal/telemetry` | Approved UDP exports and bounded resolver/Zeek JSON logs; source-specific evidence/unknown states |
| `cmd/server`, `internal/server`, `internal/auth` | Operator API, Supabase identity/session checks, enrollment, native mutual TLS and signed policies |
| `internal/store`, `internal/domain`, `supabase/migrations` | Tenant/site authorization, FORCE RLS, evidence, exact counter mathematics, durable receipts and scoped maintenance |
| `internal/archive`, `internal/analytics`, `internal/pipeline` | Immutable encrypted objects, bounded analytics projection/query, durable scoped worker coordination |
| `cmd/worker` | Separately credentialed retention, rule evaluation, notification/export delivery, analytics projection, compaction and privacy maintenance processes |
| `internal/rules`, `internal/support` | Durable rule semantics and diagnostics built from an explicit field allowlist |
| `internal/delivery`, `internal/exportarchive`, `internal/inventoryexport` | Fenced notification/export workers, encrypted private inventory snapshots and positive download DTOs |
| `internal/observability` | Fixed API/worker process measurements and bounded sampled traces without customer/request content |
| `apps/web` | React dashboard with memory-only sessions and real API data |
| `scripts/windows` | Signed installation/update, firewall and native test/recovery harnesses |
| `deploy` | Separate staging API, native TLS collector ingress and maintenance worker configurations |

## Build and test

Use Go 1.27.1 and the committed module sums. Node dependencies use the frontend lockfile. Run from the project root:

```sh
go test -race ./...
go build ./cmd/server ./cmd/worker
go build -o build/keygen ./cmd/keygen
GOOS=windows GOARCH=amd64 go build -trimpath -o build/collector.exe ./cmd/collector
cd apps/web
npm ci
npm test
npm run build
npm run test:e2e
```

For repeatable real database checks, Docker must be available. The fixture scripts own only their labeled synthetic resources:

```sh
python3 deploy/clickhouse/fixture.py start
PAZNETOPS_TEST_CLICKHOUSE_URL=http://127.0.0.1:18123 python3 supabase/tests/bootstrap.py --test --restore
python3 deploy/clickhouse/fixture.py test
python3 deploy/clickhouse/fixture.py stop
```

PostgreSQL integration tests need an **isolated** database whose name starts with `paznetops_test`. The fresh bootstrap applies all migrations, provisions random separate runtime/maintenance credentials in protected ignored files, tests real API/mTLS persistence, and restores into a second isolated database. It removes only its own temporary container and anonymous data volume by default. Direct test execution also accepts `PAZNETOPS_TEST_DATABASE_URL`, `PAZNETOPS_TEST_ADMIN_DATABASE_URL` and `PAZNETOPS_TEST_MAINTENANCE_DATABASE_URL`, or ignored `.cache/db/*-dsn` files. Never point these tests at customer data. Missing database configuration explicitly skips those cases.

The [CI workflow](.github/workflows/validation.yml) runs backend, database/analytics, dependency, Windows API and frontend checks using pinned actions. It is prepared locally; no GitHub run or remote repository is claimed. A Windows Server CI runner is useful native API evidence and does not replace dedicated Windows10/11 service acceptance.

The portable collector tests use an explicit in-memory test key protector and loopback networking. The shipped Windows service selects Windows DPAPI and native network APIs; there is no automatic non-Windows production fallback.

## Configure and deploy

Start with [deployment setup](docs/DEPLOYMENT.md), [first Windows/UniFi site acceptance](docs/FIRST_SITE_LAB.md), [archive/analytics operations](docs/ARCHIVE_PIPELINE.md), [the API contract](docs/API_CONTRACT.md), and [frontend deployment details](apps/web/DEPLOYMENT.md). Keep secrets in protected local files or hosting secret stores. The browser receives only the three public configuration values named in its deployment guide.

Use a new Supabase project and separate PazNetOps staging resources. The API runtime database role is not a table owner, superuser or RLS-bypass role. The maintenance worker has a separate credential and cannot impersonate customer users. Private security material is generated deliberately with `keygen`; process restarts never create a replacement authority silently.

The operator/enrollment listener uses verified public HTTPS at ingress. The collector listener terminates TLS itself and requires a verified client certificate plus a live database assignment on every request. It must receive raw TCP rather than a TLS-terminating proxy.

## Site visibility

A normal Ethernet connection supports permitted management queries and probes. It does not supply other clients' unicast packets. Traffic visibility requires an approved gateway export or a suitable independent metadata source. DNS names require an approved resolver/analyzer log; encrypted DNS or an unseen path remains outside coverage. No packet driver, Nmap, Npcap, Diode server or native Windows Zeek package is bundled into the baseline.

Policies contain local credential references such as `local-secret:controller` and `local-trust:controller-ca`, not credential values. Site subnet exclusions and a selected management interface are enforced again at connection/listener time. Full device configuration backup, arbitrary SSH commands and continuous packet capture are outside the baseline.

Operator identity review groups whole source records through bounded administrator previews and reversible assertions. Original source devices, interfaces, counters, graph endpoints and incident identities remain available. Retained LLDP evidence has a separate bounded observation view when local device/port association is unresolved. The typed resolver supports explicit mappings and reciprocal reports; verified agent epochs, cross-profile associations, physical classifications and firmware fixtures are still needed for automatic physical graph resolution. See [topology resolution](docs/TOPOLOGY_RESOLUTION.md) and [the API contract](docs/API_CONTRACT.md).

The signed SNMP `inventory` profile collects reported device identity, IF-MIB interfaces and unresolved LLDP under one configured source and connection. Shared call/datagram/row/byte/deadline limits preserve partial results without asserting absent devices or physical cables. The editor accepts implemented profiles and requires explicit correction of unsupported saved values. The actual portable collector, encrypted spool, mutual TLS and PostgreSQL path has been tested with an owned read-only UDP fixture; this does not establish real UniFi or Windows compatibility. See [SNMP collection](docs/SNMP_COLLECTION.md) for the collection contract and remaining model gates. The explicit classic/Q-BRIDGE endpoint profiles add independent bridge/VLAN mappings and IP neighbor reports; scoped lookups preserve acquisition boundaries and candidate ambiguity. See [Endpoint learning evidence](docs/ENDPOINT_LOCATION.md).

See the original plan for the evidence needed before claiming model support, rates, complete topology, exact traffic accounting, disaster recovery or production reliability.
