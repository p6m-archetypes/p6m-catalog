# p6m-catalog

The p6m archetype catalog: every service archetype, grouped by language and shape.

```
archetect ls p6m                       # walk it
archetect p6m/java/services/rest       # render a leaf
```

Baseline `main`. All implementation lives on the long-running `dev` branch and lands here via a
reviewed pull request (SOC 2 change management).

## The answer layers

Since YP6M-3424 the archetypes ask for **one project name** rather than an author × org × solution
× prefix × suffix decomposition (`prova-p6m-standards` docs/standards.md S1). Conventions did not
disappear with those prompts — they moved into the answer layers, which hold them harder. Archetect
resolves answers `config → -A files → -a flags`, and an answered key **suppresses its prompt
entirely**, so a supplied convention costs zero prompts and cannot be typed wrong. A free-text
project prefix never guaranteed that.

There are three layers, and what belongs in each is a rule, not a preference:

| Layer | Holds | Because |
|---|---|---|
| **This catalog** | what is true for **every customer** | it is shared across all of them |
| **Ybor Studio** | per-organization answers, injected at render time | it knows which organization the user is creating an application in |
| **The person rendering** | the project name, the domain entity | nobody else can know those |

### What this catalog encodes

`use_defaults: ["service_port", "management_port"]` on every service leaf. The p6m port convention
(8080/8081, or 50051/50052 for gRPC) is identical for every customer, and the platform manifests
inject it regardless — so taking the archetype's defaults here costs a customer nothing and removes
two prompts from every render that comes through the catalog.

### What this catalog deliberately does NOT encode

**No `answers:` for identity or infrastructure.** `organization_name`, `image_registry`,
`artifactory_host` and Java's `group_id` are all customer-specific. Putting one customer's value
here would hand it to every other customer as a default — silently, and in a file none of them
read. The catalog is shared; anything customer-shaped belongs in the layer that knows the customer.

### The contract Ybor Studio injects

Studio knows which **organization** an application is being created in, so it supplies the keys that
follow from that. These are the same keys a `-A answers.yaml` file would carry for a team rendering
from the CLI:

| Key | What it is |
|---|---|
| `organization_name` | the GitHub organization; prefixes the Kubernetes namespace as `{organization}-{application}-{env}`, which is how the platform operator reads organization and environment back out. Until YP6M-3798 it was `solution_name`, which the archetypes still accept |
| `image_registry` | the container registry the organization publishes to; the image path is `{registry}/{organization}/{application}` |
| `artifactory_host` | **java only** — the JFrog host; the Maven repositories derive as `{organization}-libs-release` / `-libs-snapshot` |
| `group_id` | **java only** — defaults to the organization with its separator swapped (`acme-payments` → `acme.payments`), so it is usually left to default |

Everything else is the renderer's own decision: `project_name`, `entity_name` (defaulted off the
project name), the persistence/cache/messaging selections, and `scm_provider`.

## Related

- [prova-p6m-standards](https://github.com/p6m-archetypes/prova-p6m-standards) — the standard the
  archetypes are held to; S1 states the identity contract, S1b the prompt-surface interface.
- [p6m-identity-library](https://github.com/p6m-archetypes/p6m-identity-library) — the single
  implementation of the identity prompt surface every archetype composes.
