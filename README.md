[![Build Status](https://damauri.visualstudio.com/azure-cli-script/_apis/build/status/azure-cli-script-CI-Cov)](https://damauri.visualstudio.com/azure-cli-script/_build/latest?definitionId=4)

# Azure CLI Script

A script language created from AZ CLI commands to make deployment and management of Azure resources as simple and intelligent as possibile. It helps to reduce by 40% the code you need to write!

## Goal

For this first realease the goal is to support all available AZ CLI commands and:

- remove all redundancy: no need to specify the same options again and again. 
- keep command signature and options consistent across all commands: in some AZ commands the resource name is not specified using the --name option. AZ  Script will correct that.
- provide context-aware environment so every command knows what happened before and can act accordingly

For example, if you want all your resources deployed into the 'eastus' region you can just write

```
location use 'eastus';
```

and all subsequent command will use that location, if not explicity overridden in the command itself. **Same logic can be applied to any resource that depend on other resource.** Storage accounts for example, or VPNs.

>Note: This is an experiment created by a customer facing team at Microsoft called, Commercial Software Engineering. We work with customers on a daily basis and as a result of that work we create this project. It is being used by partners today around the globe. Please help us improve it by trying it out and providing us feedback.

## Language

The language has been created so that you can reuse everything you already know from AZ CLI. For example to create a storage account with AZ CLI you would type something like

```
az group create -n 'dmk1' -l 'eastus'
az storage account create -g 'dmk1' -n 'dmk1storage' -l 'eastus' --sku 'Standard_LRS'
az eventhubs namespace create -g 'dmk1' -n 'dmk1ingest' -l 'eastus' --sku 'Standard' --capacity 20
az eventhubs eventhub create -g 'dmk1' -n 'dmk1ingest-32' --message-rention 1 --partition-count 32 --namespace-name 'dmk1ingest'
az eventhubs eventhub consumer-group create -g 'dmk1' -n 'cosmos' --eventhub-name 'dmk1ingest-32' --namespace-name 'dmk1ingest'
```

with AZ Script you would write

```
location use 'eastus';

group create 'dmk1';

storage account create 'dmk1storage' (
	sku: 'Standard_LRS'		
);

eventhubs namespace create 'dmk1ingest' (
	sku: "Standard",
	capacity: 20
);

eventhubs eventhub create 'dmk1ingest-32' (
	message-retention: 1,
	partition-count: 32
);

eventhubs eventhub consumer-group create 'cosmos';
```
 
isn't that so much better?

## Extensibility

AZ Script is written in Python and can easily be extended to support any kind of Azure resource just by writing a simple plugin, which is nothing more than a class derived from the base class ```Handler```. Really easy!.

The (somehow) supported resources, right now, are

- appservice
- cosmosdb
- eventhubs
- extension
- functionapp
- iot
- resource group
- storage

More will come in near future, stay tuned.

## Install

Just use the usual `pip` tool:

	pip install azsc

Done. You may want to take a look at the samples folder to get started with Azure Script:

[Azure Script Samples](./samples)

## Usage

Just run the `azsc` compiler, passing the script file you want to compile.

```
azsc <filename.azs> [--debug]
```

will generate the AZ CLI commands needed to do what defined in the script file.
`--debug` will also print the parse tree for debugging purposes

As a starting sample you can use the `e2e-1.azs` script:

	azsc .\samples\e2e-1.azs

it will generate AZ CLI script ready to be executed in a [WSL](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux) bash.

## Compilation

The result of compiling is, at present time, a transpilation to AZ CLI commands. The entire process is completely extensibile, so in future plugins to generate ARM templates or even direct REST API calls could be created.

## Notes

Grammar definition is done using EBNF format and the parses is [Lark](https://github.com/lark-parser/lark)

## Roadmap

For now this is just an experiment. Let's see where it goes...

But if you're curious here's something I have in mind:

- Support syntax highlighting Visual Studio Code (done: https://github.com/yorek/azure-script-vscode)
- Support for and code completion in Visual Studio Code
- Build a graph of dependencies and the run as many AZ CLI commands in parallel as possibile
- Using the dependency graph, validate the command even before running them
- Define a clever way to deal with erros like:
	- automatic retry 
	- break the scripts
	- take compensating actions
- Generate Powershell Script
- Generate ARM template
- Execute the commands instead of just generating a script
