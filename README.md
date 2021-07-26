# Axway Open Banking Docs

## Content

### Structure

All content is created in the `content/en/docs` directory as shown below.

```
content
| -- en
|    | -- docs
|    |    | -- deployment
|    |    | -- overview
|    |    |    | -- integration
|    |    |    | -- technical
|    |    | -- reference
|    |    |    | -- brazil
```

To explain the directories:

| Directory | Purpose |
| --------- | ------- |
| `content/en/docs` | Content "homepage". Shows overview of open banking and puts solution in context. |
| `content/en/docs/deployment` | Deployment guides. |
| `content/en/docs/overview` | Overview of solution and features. |
| `content/en/docs/overview/integration` | Integration overview with sequence diagrams showing interactions between system components. |
| `content/en/docs/overview/technical` | Technical architecture overview with information on Kubernetes architecture and a high-availability implementation. |
| `content/en/docs/reference` | Reference information not directly related to solution functionality. |
| `content/en/docs/reference/brazil` | Reference information on the Brazil market. |

### Text and Images

The process for adding new content to an existing directory is simple:

* Add a file to the relevant directory. For example, to add a new integration subpage create a file called `example.md` in `content/en/docs/overview/integration`
* Add Front Matter to the page as described below.
* Below the Front Matter add content in Markdown format.
* To add an that cannot be referenced remotely copy the image to the `static/Images`.
* To add a sequence diagram follow the guidelines in the [PlantUML](#plantuml) section.

To create *new content in a new directory*:

### PlantUML

All sequence diagrams are written using [PlantUML](https://plantuml.com/sequence-diagram) syntax. This is a text-based language that is transformed to a display format by the PlantUML Java Library.


### Static Assets

Static assets support the site functionality. There are a number of customized in the site.

#### Layouts

##### `sequence`

##### `bigtable`

### Front Matter and Markdown





## Lifecycle

### Outline

To date this project implements [GitHub Flow](https://guides.github.com/introduction/flow/), which is a simple and easy-to-use method doing documentation updates.

The outline process is as follows:

* The relevant subject matter expert creates a new branch and starts work on their changes.
* When they are happy they commit and push to the branch.
* They then create a Pull Request, which creates a Preview site that can reviewed on Netlify (link to the site is shown in the Pull Request page). The technical writer then reviews and suggest changes, etc.
* When approved the changes are merged to `master`. This kicks off a deployment in Netlify to the [Netlify site](https://axway-open-banking-docs.netlify.app/).
* The final step is a merge to production, which does not necessarily need to be performed at each merge to `master` on GitHub. This happens through a manual push to Zoomin, the production provider for https://docs.axway.com.

### Production