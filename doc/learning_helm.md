# Learning Helm

## Preface

Helm is the package manager for Kubernetes, the popular open source container management platform.

Package managers make platforms more accessible to those who use them.
In order to use a platform like Kubernetes, you need to run software on it, and much of that software will be off-the-shelf or shared.
This material may be protected by copyright.
Package managers like Helm enable you to install and start using the software quickly without needing to figure out how to make it run or run well on the platform, because it has already been packaged up in an easy-to-use manner.

## Chapter 1. Introducing Helm

Helm is the package manager for Kubernetes.

### The Cloud Native Ecosystem

#### Containers and Microservices

At the very heart of cloud native computing is this philosophical perspective that *smaller discrete standalone services* are preferable to *large monolithic services* that do everything.

#### Schedules and Kubernetes

### Helm's Goals

When we wrote Helm, we had three main goals:
* Make it easy to go from "zero to Kubernetes"
* Provide a package management system like operating systems have
* Emphasize security and configurability for deploying applications to Kubernetes

#### From Zero to Kubernetes

#### Package Management

As we envisioned Kubernetes as an operating system, we quickly saw the need for a Kubernetes package manager.

#### Security, Reusability, and Configurability

### Helm's Architecture

#### Kubernetes Resources

We have had a look at several kinds of Kubernetes resources.
We saw a couple of `Pod` definitions, a `ConfigMap`, a `Deployment`, and a `Service`.
There are dozens more provided by Kubernetes.
You can even use custom resource definitions (CRDs) for defining your own custom resource types.
The main Kubernetes documentation provides both accessible guides and detailed API documentation on each kind.

#### Charts

In Helm's vocabulary, a package is called a *chart*.

A chart is a set of files and directories that adhere to the chart specification for describing the resources to be installed into Kubernetes.

A chart contains a file called *Chart.yaml* that describes the chart.
It has information about the chart version, the name and description of the chart, and who authored the chart.

A chart contains *templates* as well.
These are Kubernetes manifests (like we saw earlier in this chapter) that are potentially annotated with templating directives.

A chart may also contain a *values.yaml* file that provides default configuration.
This file contains parameters that you can override during installation and upgrade.

When you see a Helm chart, though, it may be presented in either unpacked or packed form.

#### Resources, Installations, and Releases

To tie together the terminology introduced in this section, when a Helm chart is installed into Kubernetes, this is what happens:
1. Helm reads the chart (downloading if necessary).
1. It sends the values into the templates, generating Kubernetes manifests.
1. The manifests are sent to Kubernetes.
1. Kubernetes creates the requested resources inside of the cluster.

When a Helm chart is installed, Helm will generate as many resource definitions as it needs.
Some may create one or two, others may create hundreds. When Kubernetes receives these definitions, it will create resources for them.

A Helm chart may have many resource definitions. Kubernetes sees each of these as a discrete thing.
But in Helm's view all of the resources defined by a chart are related.

## Chapter 2. Using Helm

Helm provides a command-line tool, named `helm`, that makes available all the features necessary for working with Helm charts.

### Installing and Configuring the Helm Client

### Adding a Chart Repository

A Helm chart is an *individual package* that can be installed into your Kubernetes cluster.
During chart development, you will often just work with a chart that is stored on your local filesystem.

But when it comes to sharing charts, Helm describes a standard format for indexing and sharing information about Helm charts.
A Helm *chart repository* is simply a set of files, reachable over the network, that conforms to the Helm specification for indexing packages.

Adding a Helm chart is done with the `helm repo add` command.
Several Helm repository commands are grouped under the `helm repo` command group:
```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Now we can verify that the Bitnami repository exists by running a second repo command:
```sh
helm repo list
```

### Searching a Chart Repository

### Installing a Package

At very minimum, installing a chart in Helm requires just two pieces of information: the name of the installation and the chart you want to install.

Therefore, Helm needs a way to distinguish between the different instances of the same chart.
So an *installation* of a chart is a specific instance of the chart.
One chart may have many installations.

When we run the `helm install` command, we need to give it an installation name as well as the chart name.
So the most basic installation command looks something like this:
```sh
helm install mysite bitnami/drupal
```
The preceding will create an instance of the `bitnami/drupal` chart, and will name this instance `mysite`.

#### Configuration at Installation Time

In the preceding examples, we installed the same chart a few different ways.
In all cases, they are identically configured. While the default configuration is good sometimes, more often we want to pass our own configuration to the chart.

There are several ways of telling Helm which values you want to be configured.
The best way is to create a YAML file with all of the configuration overrides.
For example, we can create a file that sets values for `drupalUsername` and `drupalEmail`:
```yaml
drupalUsername: admin
drupalEmail: admin@example.com
```
Now we have a file (conventionally named *values.yaml*) that has all of our configuration.
Since it is in a file, it is easy to reproduce the same installation.
You can also check this file into a version control system to track changes to your values over time.
The Helm core maintainers consider it a good practice to keep your configuration values in a YAML file.
It is important to keep in mind, though, that if a configuration file has sensitive information (like a password or authentication token), you should take steps to ensure that this information is not leaked.

Both `helm install` and `helm upgrade` provide a `--values` flag that points to a YAML file with value overrides:
```sh
helm install mysite bitnami/drupal --values values.yaml
```

There is a second flag that can be used to add individual parameters to an install or upgrade.
The `--set` flag takes one or more values directly.
They do not need to be stored in a YAML file:
```sh
helm install mysite bitnami/drupal --set drupalUsername=admin
```

In general, Helm core maintainers suggest storing configuration in *values.yaml* files (note that the filename does not need to be "values"), only using `--set` when absolutely necessary.
This way, you have easy access to the values you used during an operation (and can track those over time), while also keeping your Helm commands short.
Working with files also means you do not have to escape as many characters as you do when setting things on the command line.

### Listing Your Installations

As we have seen so far, Helm can install many things into the same cluster - even multiple instances of the same chart.
And with multiple users on your cluster, different people may be installing things into the same namespace on a cluster.

The `helm list` command is a simple tool to help you see installations and learn about those installations:
```sh
helm list
```
This command will provide you with lots of useful information, including the name and namespace of the release, the current revision number (discussed in Chapter 1, and in more depth in the next section), the last time it was updated, the installation status, and the versions of the chart and app.

Like other commands, helm list is namespace aware.
By default, Helm uses the namespace your Kubernetes configuration file sets as the default.
Usually this is the namespace named `default`.
Earlier, we installed a Drupal instance into the namespace `first`.
We can see that with `helm list --namespace first`.

When listing all of your releases, one useful flag is the `--all-namespaces` flag, which will query all of the Kubernetes namespaces to which you have permission, and return all of the releases it finds:
```sh
helm list --all-namespaces
```

### Upgrading an Installation

### Uninstalling an Installation

## Chapter 3. Beyond the Basics with Helm

### Templating and Dry Runs

When Helm installs a release, the program steps through several phases.
It loads the chart, parses the values passed to the program, reads the chart metadata, and so on.
Near the middle of the process, Helm compiles all of the templates in the chart (all in one pass), and then renders them by passing in the values (like we saw in the previous chapter).
During this middle portion, it executes all of the template directives.
Once the templates are rendered into YAML, Helm verifies the structure of the YAML by parsing it into Kubernetes objects.
Finally, Helm serializes those objects and sends them to the Kubernetes API server.

Roughly, then, the process is:
1. Load the entire chart, including its dependencies.
1. Parse the values.
1. Execute the templates, generating YAML.
1. Parse the YAML into Kubernetes objects to verify the data.
1. Send it to Kubernetes.

The generated values object is created by loading all of the values of the chart file, overlaying any values loaded from files (that is, with the `-f` flag), and then overlaying any values set with the `--set` flag.
In other words, `--set` values override settings from passed-in values files, which in turn override anything in the chart’s default *values.yaml* file.

It is important to note that, when executed, some Helm templates require information about Kubernetes.
So during template rendering, Helm may contact the Kubernetes API server.
This is an important topic that we will discuss in a moment.

The output of the preceding step is then parsed from YAML into Kubernetes objects.
Helm will perform some schema-level validation at this point, making sure that the objects are well-formed.
Then they will be serialized into the final YAML format for Kubernetes.

In the last phase, Helm sends the YAML data to the Kubernetes API server.
This is the server that kubectl and other Kubernetes tools interact with.

The API server will run a series of checks on the submitted YAML.
If Kubernetes accepts the YAML data, Helm will consider the deployment a success.
But if Kubernetes rejects the YAML, Helm will exit with an error.

Later on, we'll go into detail about what happens once the objects are sent to Kubernetes.
In particular, we'll cover how Helm associates the process described earlier with an installation and revisions.
But right now, we have enough information about workflow to understand two related Helm features: the `--dry-run` flag and the `helm template` command.

#### The `--dry-run` Flag

Commands like `helm install` and `helm upgrade` provide a flag named `--dry-run`.
When you supply this flag, it will cause Helm to step through the first four phases (load the chart, determine the values, render the templates, format to YAML).
But when the fourth phase is finished, Helm will dump a trove of information to standard output, including all of the rendered templates.
Then it will exit without sending the objects to Kubernetes and without creating any release records.

The principal purpose of the `--dry-run` flag is to give people a chance to inspect and debug output before sending it on to Kubernetes.
But soon after it was introduced, Helm maintainers noticed a trend among users.
People wanted to use `--dry-run` to use Helm as a template engine, and then use other tools (like `kubectl`) to send the rendered output to Kubernetes.

To remedy these problems, the Helm maintainers introduced a completely separate command: `helm template`.

#### The `helm template` Command

While `--dry-run` is designed for debugging, `helm template` is designed to isolate the template rendering process of Helm from the installation or upgrade logic.

### Learning About a Release

#### Release Records

#### Listing Releases

#### Find Details of a Release with `helm get`

While `helm list` provides a summary view of installations, the `helm get` set of commands provide deeper information about a particular release.

There are five `helm get` subcommands (`hooks`, `manifests`, `notes`, `values`, and `all`). Each subcommand retrieves some portion of the information Helm tracks for a release.

### History and Rollbacks

#### Keeping History and Rolling Back

In the previous chapter, we saw that the helm uninstall command has a flag called `--keep-history`.
Normally, a deletion event will destroy all release records associated with that installation.
But when `--keep-history` is specified, you can see the history of an installation even after it has been deleted:
```sh
helm uninstall wordpress --keep-history
```

### A Deep Dive into Installs and Upgrades

#### The `--generate-name` and `--name-template` Flags

This has made certain tasks a little more complicated.
For example, a CI system that automatically deploys things must be able to ensure that the name it gives these things is unique within the namespace.
One approach to dealing with this issue is for Helm to provide a tool for generating a unique name.
(Another approach is to always overwrite a name if it already exits.
See the next section for that approach.)

Helm provides the `--generate-name` flag for `helm install`:
```sh
helm install bitnami/wordpress --generate-name
```

#### The `--create-namespace` Flag

## Chapter 4. Building a Chart

Charts are at the heart of Helm.
In addition to installing them into a Kubernetes cluster or managing the instances of charts you've installed, you can build new charts or alter existing ones.
In the next three chapters we will cover a lot of details about charts including creating them, the elements inside them, templating Kubernetes manifests, testing charts, dependencies, and more.

In this chapter you will learn how to create a new chart and learn about the many parts of a chart.
This will include the use of several built-in commands that can help you in the chart development process.

Charts are the packages Helm works with.
They are conceptually similar to Debian packages used by APT or Formula used by Homebrew for macOS.
The conceptual similarity is where the similarities end.
Charts are designed to target Kubernetes as a platform that has its own unique style.
At the heart of charts are templates to generate Kubernetes manifests that can be installed and managed in a cluster.

### The Chart Creation Command

Helm includes the `create` command to make it easy for you to create a chart of your own, and it's a great way to get started.
This command creates a new Nginx chart, with a name of your choice, following best practices for a chart layout.
Since Kubernetes clusters can have different methods to expose an application, this chart makes the way Nginx is exposed to network traffic configurable so it can be exposed in a wide variety of clusters.

The `create` command creates a chart for you, with all the required chart structure and files.
These files are documented to help you understand what is needed, and the templates it provides showcase multiple Kubernetes manifests working together to deploy an application.
In addition, you can install and test this chart right out of the box.

### The Chart.yaml File

You might notice that the style of a *Chart.yaml* file is similar but mildly different from those of Kubernetes manifests.
*Chart.yaml* files are not the same format as custom resources but do contain some of the same properties.
The original *Chart.yaml* files were designed back in 2015, long before Kubernetes custom resource definitions existed.
While Helm has progressed in major versions, it has maintained a certain amount of backward compatibility over time to not disrupt users too much.
This has led to differences between the *Chart.yaml* file format and Kubernetes manifests.

### Modifying Templates

Helm is written in the Go programming language, and Go includes template packages.
Helm leverages the text template package as the foundation for its templates.
This template language is similar to other template languages and includes loops, if/then logic, functions, and more.
An example template of a YAML file follows:
```go
product: {{ .Values.product | default "rocket" | quote }}
```
In this YAML file there is a key name of `product`.
The value is generated using a template.
`{{` and `}}` are the opening and closing brackets to enter and exit template logic.
There are three parts to the template logic separated by a `|`.
This is called a pipeline, and it works the same way as a pipeline in Unix-based systems.
The value or output of a function on the left is passed in as the last argument to the next item in the pipeline.
In this case, the pipeline starts with the value from the property in `.Values.product`.
This comes from the data object passed in when the templates are rendered.
The value of this data is piped as the last argument to the `default` function, which is one of the functions provided by Helm.
If the value passed in is empty, the `default` function uses the default value of "rocket", ensuring there is a value.
This is then sent to the `quote` function, which ensures the string is wrapped in quotes before writing it to the template.

The `.` at the start of `.Values.product` is important.
This is considered the root object in the current scope.
`.Values` is a property on the root object.

#### The Deployment

The `include` template function enables including the output of one template in another template, and this works in pipelines.
The first argument to the `include` function is the name of the template to use.
The `.` passed in as the second argument is the root object.
This is passed in so the properties and functions on the root object can be used within the called template.

`anvil.fullname` and `anvil.labels` are two reusable templates included in the chart via the *_helpers.tpl* file.
(The `_` at the start of the name causes it to bubble up to the top of directory listings so you can easily find it among your templates; Helm does not render them into Kubernetes manifests but does make templates in them available for use.)
`anvil.fullname` provides a name based on the name chosen when the chart is instantiated, and `anvil.labels` provides labels following Kubernetes best practices.
The functions are covered in more depth in Chapter 5.

### Using the Values File

Charts include a *values.yaml* file that sits alongside the *Chart.yaml* file in the root of a chart.
The *values.yaml* file contains the default values used by the chart, and it is a form of documentation for the custom values that can be passed into a chart.

*values.yaml* is an unstructured YAML file.
There are some common and useful practices, which will be covered shortly, but nothing is required in the format of the YAML.
This enables chart creators to provide a structure and information that works well for them.
A *values.yaml* file can contain numerous things, from simple substitution for Kubernetes manifest properties to elements needed for application-specific business logic.

#### Container Images

#### Exposing Services

#### Resource Limits

### Packaging the Chart

Helm has the ability to build a chart archive.
Each chart archive is a gzipped TAR file with the extension *.tgz*.
Any tool that can create, extract, and otherwise work on gzipped TAR files will work with Helm's chart archives.

When Helm generates the archive files, they are named using a pattern of *chart name-version.tgz*.
Helm expects this same pattern when consuming them.
The *chart name* is the name you will find inside the *Chart.yaml* file and the *version* is the chart version.
This enables multiple versions of the same chart to be stored alongside each other.
You can package Anvil as an archive by running:
```sh
helm package anvil
```

### Linting Charts

When developing charts, especially when working with YAML templates, it can be easy to make a mistake or miss something.
To help you catch errors, bugs, style issues, and other suspicious elements, the Helm client includes a linter.
This linter can be used during chart development and as part of any testing processes.

To use the linter, use the `lint` command on a chart as a directory or a packaged archive:
```sh
helm lint anvil
```
