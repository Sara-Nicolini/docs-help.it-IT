
# External Docs Contributor Guide 

This repo provides publishing content for the [external version of the Docs Contributor Guide](https://docs.microsoft.com/contribute), but not all of the content in the guide is sourced from this repo. Content used for build/publication of the external guide is sourced from the following locations:

1. This repository (github.com/microsoftdocs/docs-help.git) - provides the following 4 items: 

   - main landing page (help-content/index.md)
   - Table of Contents (help-content/TOC.md)
   - breadcrumb configuration (help-content/breadcrumb/toc.yml)
   - custom 404 page (help-content/404.md)
 
   In addition, as an OPS repository, all required build/publishing configuration files are housed in this repository. 

2. [The Docs Help guides repository](https://github.com/MicrosoftDocs/docs-help-pr/), which is the source for core Docs contributor guide content, both internal and external, along with content for the Style and Onboarding guides. Selected content is pulled for publication into the external contributor guide, using the [OPS Cross Repository Reference (CRR) feature](https://opsdocs.azurewebsites.net/en-us/opsdocs/partnerdocs/cross_repository_reference?branch=master). Note that the internal contributor guide is published only to staging at https://review.docs.microsoft.com/en-us/help/contribute?branch=master.

3. [The OPS user guide repository](https://github.com/MicrosoftDocs/openpublishing-docs), which is used by the core contributor guide to pull selected articles, also via CRRs. Note that the OPS user guide is also internal only, and is published to https://opsdocs.azurewebsites.net/en-us/opsdocs/index?branch=master.

## Publishing changes to the external contributor guide

### Guidelines

Per the previous section, all shared Markdown files live in the docs-help-pr repo. When you are editing shared Markdown files (ie: ones that publish to both the internal and external guides), please consider the following.

- Don't add content that should not be published externally
- Don't add links to other internal-only pages
- Wherever possible, refrain from using internal acronyms such as C+E, APEX, etc.

### Special PR considerations

Like most other OPS repositories, publishing to the release site (docs.microsoft.com) is done by merging a repo's 'master' branch commits into the 'live' branch. But because the external contributor guide is heavily reliant on CRRs, the normal pull request mechanism may not work the way you would expect. This is because:  

- a GitHub PR will not detect changes in content pulled in via CRR, because it lives in a different repository. The only changes that will be detected by a pull request in this repository, are changes to the files that live in this repo (listed under #1 in the section above).  

- only CRR files that are new/added will be pulled for publication, not updated .md files, due to current OPS CRR limitations. A feature request is in the works (see work item [1029304](https://mseng.visualstudio.com/CSI/_workitems/edit/1029304) for details) that will enable modified files to be pulled as well.  
    
In order to publish content to the external site, you must:

- create a 'master' to 'live' PR that is accompanied by changes/commits to files that reside in this repository, before it will pull all of the files referenced via CRR. This can be as simple as adding a space to a file, if you don't have any necessary changes. If you simply try to create a PR from 'master' to 'live', with no related changes to files in this repo, GitHub will not detect any differences between the 2 branches (rightly so), and will therefore not allow you to create a new PR. Once the PR has built successfully, you can merge into the 'live' branch.

- in addition, after the PR merges, you must do a "forced" publish on the 'live' branch using the [OPS publishing portal](https://op-portal-prod.azurewebsites.net/#/containers/history/repositories/All):

   - Select "Repository Management"
   - Search for the MicrosoftDocs/docs-help repo 
   - Select the row (not the repo link), which will cause the row to expand and show the docset -> endpoint row
   - Select the "Publish" button at the top of the page
   - Then select the branch, the "Publish" target, and "Force Publish", then click Publish

   This will cause a forced publication of all files from the branch specified. Note that this must be done for any branch you are testing (ie: a temporary "pr-en-us-nn" branch, 'master', 'live', etc.).

## Additional details on Cross Repository References (CRRs)

You will find the configuration info for the CRRs mentioned above, in the [dependent_repositories section of the .openpublishing.publish.config.json file](.openpublishing.publish.config.json). As of the time of this writing, there were 2 CRRs configured :

  - help-content/help-crr : refers to content stored in the [Docs help guide repository](https://github.com/MicrosoftDocs/docs-help-pr/)
  - help-content/ops-crr : refers to content stored in the [OPS user guide repository](https://github.com/MicrosoftDocs/openpublishing-docs)

References to content in the above repositories can be achieved by using the CRR's `path_to_root` property, plus the path to the article file within the respective repository, in a relative link reference. The `path_to_root` properties contains the external contributor guide's main docset name (`help-content`), plus a unique identifier for each CRR (`help-crr` and `ops-crr`). For example, the TOC entry `[Metadata reference](~/help-crr/help-content/contribute/contribute-how-to-write-metadata.md)`, will cause the page located at https://github.com/MicrosoftDocs/docs-help-pr/blob/master/help-content/contribute/contribute-how-to-write-metadata.md to be published for that link. 

### Exclusion of CRR content via docfx.json

You can (and should) use the `build`/`content`/`exclude` element of the [docfx.json file](https://github.com/MicrosoftDocs/docs-help/blob/master/help-content/docfx.json) to control the portions of the CRR repository content you would like to exclude during publishing. This is also useful to avoid build warning/error noise, caused by problems from trying to build unnecessary/unrelated files located in CRR repos (see below). If you don't use this exclude feature during build, you will essentially pull in the entire CRR repository, whether you need all of the content or not, so it also limits the scope of the build/publish, improving performance.

### Debugging PR warnings/errors caused by CRRs

Many times you will get PR build warnings/errors, that are triggered by problems in files within a repo referenced via CRR. Normally these will seem like false positives, because they are not files that exist in this repository. And in fact, clicking on the linked file in the error message may result in a 404. 

These are usually linkage errors, or build errors that are specific to the CRR file in question, and typically fall within 2 categories:

1. Files you don't care about, because you either don't need to build/publish them as part of this repo's publication, or they've been intentionally excluded from publication to prevent access by external users.
2. Files you may care about, which may be necessary for the build/publish cycle of this repo. These typically become more prominent during the master-to-live merge.

For those falling under #1:  

- if you're getting build errors caused by CRR content that you don't need anyway (ie: linkage problems within the CRR repo files), you can usually remove the files (or entire subdirectory) using the technique described under [Exclusion of CRR content via docfx.json](#exclusion-of-crr-content-via-docfxjson). 
- if you're seeing link warnings, they are likley caused by intentional removal of internal guide content, which you can safely ignore. For instance, warnings that start with `[Warning] Invalid file link:(~/ops-crr/openpublishing/docs/gauntletdocs ...` can be ignored, as the external guide does not publish the gauntlet extension docs. This is an example of content that was filtered from the TOC, but also needed to be removed from publishing as it is linked to from several places (a better alternative than trying to write 2 different pages for each place that links to this content). There are also cases where files have explicitly been excluded from publication, if they are linked to from a file that needs to be published. In all cases though, the 404.md file will provide a more user-friendly error response.

For those falling under #2, you will need to do one of the following : 

- If it's a file you know you need, submit a PR or work with the CRR repo owner to get the problem resolved. For example, a build warning such as `[Warning] Invalid file link:(~/ops-crr/openpublishing/docs/partnerdocs/configuring-scoped-search.md)`, indicates that the problem is in the repo specified by the `ops-crr` CRR, under the `openpublishing/docs/partnerdocs/configuring-scoped-search.md` path within that repo. These can also be temporary problems, where the build cycle from a PR in this repo is picking up bugs from the CRR repo, which eventually get fixed. 

- Ignore the warning. This seems counter intuitive, but frequently this is the best/easiest action. This is mainly caused when a file you DON'T need comes into this repo's build cycle as part of a larger group of files you DO need (ie: under a subdirectory in the CRR repo), and it links to a file/files you are intentionally excluding from the build. The only other solution is to do a more granular [file-by-file exclusion](#exclusion-of-crr-content-via-docfxjson), if the error/warning becomes annoying.
