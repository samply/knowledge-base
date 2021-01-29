# Adapting and Extending the FHIR IG

This guide assumes you have a basic understanding of FHIR and profiling. If not, the [FHIR DevDays Youtube channel](https://www.youtube.com/channel/UCH1DB1ZJbUdC5DOn110lZ_Q) is a good place to start.

## Background

This guideline is intended for developer / data modelers / everyone wanting to extend, adapt or exchange the underlying data model of the Sample Locator. The process of adapting the GUI and queries etc. is not documented yet (but some team members are familiar with it, ask someone from the DKFZ team).
This document will only focus on the means to communicate your changes / new model to those responsible for filling the local data store.
In FHIR, an instruction on how to build resources for a specific use case is called an *Implementation Guide* (IG). An IG usually consists of text explaining the use case and how FHIR plays a role,
profiles that describe how FHIR resources should look like in the context of this project ( if you are not familiar with what FHIR profiles are, read e.g. the introduction [here](https://www.hl7.org/fhir/profiling.html) ) and some terminology resources like ValueSets, ConceptMaps or CodeSystems that tell implementers which codes to use when in the context of the project covered by the IG.

## How to extend the data model

It may become necessary to extend the data model, e.g. to add a new data elements that can be searched for. In this case, you should first answer the following questions:

1. *Is the addition part of the core GBA IG, the Oncology IG (see below), or should it be a new module?* Is it something all or most participating biobanks must or may want to use, or is it only relevant for a subgroup / subproject? In case of subprojects, you may want to create a new IG that depends on the core GBA IG and has profiles that inherit from the core profile (for more info on profile inheritance, read the corresponding section below or in [the FHIR spec](https://www.hl7.org/fhir/profiling.html#reslicing)).
2. *Do other IGs already cover this scope?* A web search for "FHIR + Topic" may find work that other people already have done on this subject. If nothing turns up, ask on the #implementers stream of chat.fhir.org (accounts are free and a must have for everyone working with FHIR). If you find a fitting IG thats seems well-maintained, use it e.g. by adding a short section with a description (e.g. which specifics profiles out of the IG to use when)  and a link to the core IG. Also add a formal dependency to the package the IG corresponds to (in Simplifier: Package Tab of project corresponding to the IG, with IG Publisher Templates the Package Id and version are usually in the website footer) to the core IG (see [SUSHI documentation](https://fshschool.org/docs/sushi/configuration/#recommended-configuration)). Using parts of other IGs instead of defining your own profiles helps interoperability and is in the spirit of FHIR (and also usually less work).

## SUSHI

SUSHI is a tool that converts profile / resource definitions written in FHIR Shorthand (FSH) into actual JSON FHIR resources. Using FSH comes with [many advantages](http://build.fhir.org/ig/HL7/fhir-shorthand/#relationships-to-other-standards-tools-and-guidelines). As an additional bonus, SUSHI greatly simplifies creating IGs for the IG Publisher by creating the numerous configuration files the publisher needs out of a single SUSHI configuration file. If you need to create a new IG from scratch, there is also an `--init` command that will create a bare-bones IG to start from. If you are not familiar with SUSHI yet, there is a [tutorial on the SUSHI website](https://fshschool.org/docs/tutorials/basic/).

## IG Publisher

The IG Publisher is developed by HL7. Its code and releases can be found [on Github](https://github.com/HL7/fhir-ig-publisher). The IG Publisher takes images, resources, profiles, text... and builds a webpage for your IG. This process is guided by numerous configuration files and an actual [ImplementationGuide resource](https://www.hl7.org/fhir/implementationguide.html). Luckily, SUSHI can create this IG resource instance and configuration files for you (see section on SUSHI), so you only need to worry about in which subfolders of `input` to put which files (The SUSHI team also provides some examples/guidance in their documentation).
There is also a [confluence page with documentation](https://confluence.hl7.org/display/FHIR/IG+Publisher+Documentation) on the publisher. If you still cannot get something to work, it is best to ask on the FHIR chat (chat.fhir.org, streams e.g. #tooling or #IG creation). Since the publisher is still under very active development, you might have just found a bug.
The way the generated website looks (logos, colors) is specified by a template. This template is specified in one of the configuration files or, if using SUSHI, in the SUSHI configuration file (see SUSHI documentation). The [Guidance IG](https://build.fhir.org/ig/FHIR/ig-guidance/index.html) lists some available templates. For most projects, you can start with the base template. If you want/need a custom template, you can add a local template as a folder to your IG. For an example on how to do this, [see the GBA/BBMRI.de IG](https://github.com/samply/bbmri-fhir-ig).
The publisher can be executed as part of a CI/CD pipeline and the resulting output files deployed to a webserver of your choice. For an example on how to do this, see the GBA/BBMRI.de IG repo.

There is one problem with this approach: The package generated alongside the website is not automatically added to the FHIR package registry, meaning that users would need to download and install it manually in order to use it e.g. for validation. Adding a package to the FHIR registry involves [creating and maintaining a RSS feed](https://registry.fhir.org/submit). As an alternative solution, create a project on the Simplifier platform. For each new release, all resources are deleted from Simplifier and then the new versions are bulk-uploaded (via Upload -> From file -> Chose a .zip). Then, a new package is created via the package tab. Make sure the version is exactly the same as on the IG website you created with the publisher for this release! Since all Simplifier packages are added to the package registry, users can now get the package from there.

## The GBA/bbmri.de IG

The IG describing the core data model of the original Sample Locator data store is known as the GBA/bbmri.de IG. Its source files are available [here on Github](https://github.com/samply/bbmri-fhir-ig).
From these source files, the IG is build using the open-source tools [SUSHI](https://fshschool.org/docs/sushi/) and the [IG Publisher](https://confluence.hl7.org/display/FHIR/IG+Publisher+Documentation).
The output IG is essentially a website, currently hosted on Github pages. The package is currently created with the method described in the Publisher section. Since SUSHI is a relatively new addition to the FHIR ecosystem, not all resources in the IG are created from FSH, some exist as JSON files in the repo and are picked up by the IG Publisher directly.

## The DKTK/Oncology IG

The DKTK IG used for oncology data in GBA is managed by the DKFZ team. It can be found on Simplifier [here](https://simplifier.net/oncology). Its profiles are fully compatible with the GBA/bbmri.de profiles (instances can conform to profiles from both IGs at the same time).

## FHIR Profiling

### Using multiple profiles

One resource can adhere to multiple profiles at a time, as long as there are no conflicting rules in the profiles. This means there is not necessarily a conflict if your users need to adhere to multiple IGs.

### Inheritance

Profiles can inherit from other profiles. When defining the Profile, instead of the base resource (e.g. "Specimen"), the profile to be inherited from (e.g. "BBMRI.de Specimen") is used. The new profile then contains all rules, definitions and constrains from its parent profile and can add new ones. This is useful if you need to adhere to and extend an existing project, e.g. [the profiles from the Medical Informatics Initiative](https://simplifier.net/organization/koordinationsstellemii/~projects).
