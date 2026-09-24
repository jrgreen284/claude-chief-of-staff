---
type: config
user_name:            # e.g. Alex
timezone:             # IANA zone, e.g. Europe/London
spelling: British     # British or American
start_time: "09:00"
wrap_up_time: "17:00"
finish_time: "17:30"
confidentiality:      # e.g. "Never copy client commercial terms, personal data or raw ticket content into reviews or briefs."
folders:
  reference: "0) Reference Files"
  templates: "0) Reference Files/Templates"
  company: "1) Company"
  clients_vendors: "2) Clients & Vendors"
  tasks: "3) Tasks & Actions"
  learning: "4) Learning"
  daily: "5) Daily"
  meetings: "6) Meetings"
  reviews: "7) Reviews"
about_me: "0) Reference Files/about-me.md"
career_goals: "0) Reference Files/career-goals.md"
task_files:
  - file: work-clients.md
    label: Client work
    scope: Delivery work for clients and vendors
    id_prefixes: []             # one per client or vendor, e.g. [GLO, HOO]
  - file: work-company.md
    label: Company work
    scope: Work for your own company
    id_prefixes: [CO]
organisations: []
# e.g.
#  - name: Acme
#    kind: company                # company, client or vendor
#    folder: "1) Company/Acme"
#  - name: Globex
#    kind: client
#    folder: "2) Clients & Vendors/Clients/Globex"
projects: []
# e.g.
#  - id_prefix: GLO
#    folder: "2) Clients & Vendors/Clients/Globex/Projects"
#    auto_create: false           # true = create missing project notes from the template
#  - id_prefix: CO
#    folder: "1) Company/Acme/Projects"
#    auto_create: true
horizons:
  focus_plan_days: 14
  watch_days: 10
due_date_defaults:
  High: 3
  Medium: 7
  Low: 14
standing_items: []
integrations:
  microsoft_365: false
---
# Chief of Staff config

Edit the frontmatter above to change how the Chief of Staff agents behave. Every path is relative to the vault root. Re-run `/chief-of-staff:setup` at any time to add organisations or task files — it never overwrites existing notes.
