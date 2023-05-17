# Helm

- Helm is package manager for Kubernetes
- Helm helps in:
  - reducing / managing complexity with application Installation
  - Easily update / rollback applications
  - share charts
- Helm packages are are knows as "**charts**"
- Almost all complex applications have various objects : Deployment, Service & Secrets
- Helm wraps up all these yaml files into a single object called "Charts"
- Instance of the installed chart is called as a **release**

# Helm Charts

## Chart Structure

```bash
dhayanand@master1:/tmp/dump/mysql$ tree -L 1
.
├── Chart.lock
├── charts
├── Chart.yaml
├── README.md
├── templates
├── values.schema.json
└── values.yaml
2 directories, 5 files
dhayanand@master1:/tmp/dump/mysql$
```

- **Chart.yaml**
	- name of the chart
	- version of the chart
	- description of chart
- **values.yaml**
	- default configuration values for the chart
	- referenced in the YAML files within the templates
- **charts directory**
	- any charts that this cart might depend on
- **templates directory**
	- contains template files, that when Helm evaluates the chat will send all the files through template rendering engine to produce the manifest files
	- Files in `template` directory are treated as if they contain Kubernetes except for file starting with an underscore `_` like `_helperts.tpl`
	- Those files are assumed not to contain manifest so are not updated to be Kubernetes object definition when the cart is deployed but are made available for other templates to be used
	- These files are used to store helpers and `_helperts.tpl` is the default location
	- eg: defining function to generate labels for Kubernetes
	- `NOTES.txt` contains information about the chart like default values and guide on how to deploy chart

		```bash
		dhayanand@master1:/tmp/dump/mysql/templates$ tree -L 1
		.
		├── extra-list.yaml
		├── _helpers.tpl
		├── metrics-svc.yaml
		├── networkpolicy.yaml
		├── NOTES.txt
		├── primary
		├── prometheusrule.yaml
		├── rolebinding.yaml
		├── role.yaml
		├── secondary
		├── secrets.yaml
		├── serviceaccount.yaml
		└── servicemonitor.yaml
		
		2 directories, 11 files
		dhayanand@master1:/tmp/dump/mysql/templates$ 
		```

