# About This Site

The **watsonx Orchestrate Bible** is an authoritative technical reference and lab guide for IBM watsonx Orchestrate, synthesised from the IBM watsonx Orchestrate Deep Dive Enablement Series. It is intended for IBM field engineers and technical sellers who need precise, deep technical knowledge to design, implement, and troubleshoot production-grade agent deployments.

---

## Source Material

This site was built from the **IBM watsonx Orchestrate Deep Dive Enablement Series with Development** — a sequence of 11 recorded sessions covering the full product surface. The sessions were recorded live with IBM development team members and include slides, transcripts, demos, and supplementary resources.

| Session | Topic | Materials |
|---------|-------|-----------|
| Session 01 | ADLC (Agent Development Lifecycle) | Recording · Slides · Transcript · Sample Agents & Tools |
| Session 02 | On-Prem Install, Architecture, and Operations | Recording · Transcript |
| Session 03 | Agent and Tool Design (incl. Agentic Memory) | Recording · Slides · Demo · Transcript |
| Session 04 | Evaluation and Optimization | Recording · Slides · Transcript |
| Session 05 | Security, SSO, OBO, and Agent Identity | Recording · Transcript |
| Session 06 | Load and Performance Testing | Recording · Transcript |
| Session 07 | Observability | Recording · Transcript |
| Session 08 | WXA to WXO Migration *(out of scope — not included)* | Recording · Transcript |
| Session 09 | Agentic Workflows | Recording · Slides · Use Case Guide · Transcript |
| Session 10 | Agent Control Plane and External Agents | Recording · Transcript |
| Session 11 | Voice, Phone, and Channel Integration | Recording · Transcript |

Session 08 (WXA to WXO Migration) was deliberately excluded from this Bible — it covers a one-time migration path that is no longer broadly relevant for new deployments.

---

## How Content Is Organised

Source material was analysed and synthesised into five Books, each serving a distinct audience need:

| Book | Purpose |
|------|---------|
| **[Reference](reference/index.md)** | Authoritative specification of every concept, component, and configuration surface. Treat it like a technical standard — start here to understand *what* something is and *how* it works internally. |
| **[Operations](operations/index.md)** | Installation, sizing, monitoring, load-testing, and troubleshooting for production deployments. Covers both SaaS and On-Premises (Cloud Pak for Data) topologies. |
| **[HOWTO](howto/index.md)** | Goal-oriented task guides. One page per question: *"How do I…?"* — with numbered steps, prerequisite callouts, and direct cross-links to the Reference spec. |
| **[Cookbook](cookbook/index.md)** | End-to-end, copy-paste-ready recipes for common integration patterns. Each recipe is self-contained and annotated for production adaptation. |
| **[Labs](labs/index.md)** | A 12-lab progressive hands-on series built around a single cumulative scenario ("Acme Employee Assistant"). Lab 1 assumes zero prior experience; Lab 12 is a capstone touching every major feature. On-Premises differences are surfaced in callout boxes throughout. |

---

## Roadmap Disclaimer

!!! warning "Roadmap content may be outdated"
    Roadmap information in the [Reference → Roadmap](reference/roadmap/index.md) section reflects the product direction communicated during the enablement sessions. IBM product roadmaps are subject to change. **Always consult the [official IBM watsonx Orchestrate documentation](https://www.ibm.com/docs/en/watsonx/watson-orchestrate/current) and your IBM contact for up-to-date roadmap information.**

---

## Accuracy and Currency

All technical content was written from first-hand session recordings and transcripts. Where the sessions described behaviour that was "in development" or "coming soon", this is clearly marked in the relevant pages with `!!! note "Roadmap"` callout boxes.

This site does **not** include:

- Marketing material or product positioning copy
- Feature-level claims without technical backing from the source sessions
- Session 08 migration content (out of scope)

---

## Using This Site

- **Search** works full-text across all content — use the search bar (or press `/`) to jump directly to what you need.
- **Nav breadcrumbs** show exactly where you are in the Book → Section → Page hierarchy.
- **Cross-links** — every HOWTO, Cookbook, and Lab page links back to the relevant Reference page(s) so you can go deep without losing context.
- **On-Prem callouts** — wherever SaaS and On-Premises behaviour differ, the difference is surfaced inline in a `!!! note "On-Premises"` block rather than duplicating entire pages.
- **Last-updated dates** appear at the bottom of every page (from git history).

---

## Feedback and Contributions

This site lives in a git repository. If you find an error, a missing detail, or a broken link, open an issue or submit a pull request.
