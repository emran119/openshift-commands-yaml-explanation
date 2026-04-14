# openshift-commands-yaml-explanation
This repo contains openshift command explanations along with YAML explanations.

🧠 High-level difference
Command	Purpose	When to use
oc process	Render a template → generate YAML	Before deployment
oc apply	  Create or update (declarative)	Day-to-day updates
oc replace	Overwrite existing resource (imperative)	Force full update

🔹 1. oc process → Template Engine

✅ What it does
Takes an OpenShift Template
Replaces parameters
Evaluates things like:
${PARAM}
your template.openshift.io/expose-*
Outputs plain Kubernetes YAML

🔧 Example
oc process -f template.yaml -p DB_NAME=mydb

👉 Output:

kind: Secret
data:
  database-name: bXlkYg==
  
🚨 Important
❌ Does NOT create anything in cluster
✔ Just prints YAML (or pipes it)

💡 Common usage
oc process -f template.yaml | oc apply -f -

🔹 2. oc apply → Declarative (Smart Update)
✅ What it does
Creates resource if it doesn’t exist
Updates only changed fields
Keeps track of previous config

🔧 Example
oc apply -f deployment.yaml

🧠 Behavior
Merges changes
Preserves fields not in your YAML
Uses last-applied-configuration

💡 Think of it like:
“Make the cluster look like this file”

🔥 Best for
CI/CD pipelines
GitOps
Incremental updates


🔹 3. oc replace → Full Overwrite

✅ What it does
Completely replaces the existing resource
Requires resource to already exist

🔧 Example
oc replace -f deployment.yaml

🚨 Behavior
❌ No merge
❌ No history tracking
✔ Full overwrite
💥 Risk

If your YAML is missing fields:

spec:
  replicas: 2

👉 Everything else in spec may get wiped 😬

💡 Think of it like:

“Delete old config and put this exactly”

🔥 Side-by-side example

Suppose existing Deployment has:

replicas: 3
strategy: RollingUpdate

👉 Using oc apply
replicas: 2

✅ Result:

replicas: 2
strategy: RollingUpdate   # preserved

👉 Using oc replace
replicas: 2

❌ Result:

replicas: 2
# strategy LOST

🧩 How they work together

Typical OpenShift flow:

oc process -f template.yaml \
  -p DB_USER=admin \
  -p DB_PASS=secret \
| oc apply -f -

⚡ Quick mental model

Command	Analogy:

oc process	 “Fill the form”
oc apply	   “Update only what's changed”
oc replace	 “Rewrite everything from scratch”

🚨 When to use what (practical advice)

✅ Use oc process
When working with Templates
When parameters are involved

✅ Use oc apply (MOST COMMON)
Safe updates
CI/CD pipelines
GitOps workflows

⚠️ Use oc replace carefully
When you want a clean overwrite
When fixing drift or corruption
Not ideal for regular deployments

🧠 Pro Tip (very important)

👉 Prefer this pattern always:

oc process ... | oc apply -f -

👉 Avoid:

oc process ... | oc replace -f -

unless you really know why.
