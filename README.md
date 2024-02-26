# 🏷️❌ Tag Terminator: 🤖 Git Tag Management 

This project demonstrates how to automatically delete a Git tag if it's pushed by mistake, in the absence of tag protection rules. This is particularly useful in scenarios where you want to enforce a certain naming convention or versioning scheme for your tags.

## Overview

We use GitHub Actions to monitor the creation of tags in the repository. When a new tag is created, a GitHub Actions workflow is triggered. This workflow checks if the tag name is greater than "v12.0". If it is, the workflow deletes the tag both locally and from the remote repository.

## How It Works

The main logic is contained in a GitHub Actions workflow file. This file is located in the `.github/workflows` directory of the repository.

The workflow is triggered on the `create` event, which is fired whenever a branch or tag is created. The workflow then checks if the created ref is a tag and if its name is greater than "v12.0". If both conditions are met, the workflow deletes the tag.

Here's a simplified version of the workflow:

```github-actions-workflow
name: Delete tags greater than v12.0

on:
    create:
        tags:
            - v*

jobs:
    delete-tag:
        runs-on: ubuntu-latest
        steps:
        - name: Checkout code
            uses: actions/checkout@v2
            with:
                fetch-depth: 0  # Fetch all history for all branches and tags

        - name: Delete tag
            run: |
                TAG_NAME=${{ github.ref }}
                TAG_VERSION=${TAG_NAME#refs/tags/v}
                MIN_VERSION="12.0"

                if (( $(echo "$TAG_VERSION > $MIN_VERSION" | bc -l) )); then
                    echo "Deleting tag $TAG_NAME"
                    git tag -d ${TAG_NAME#refs/tags/}
                    git push origin :${TAG_NAME#refs/tags/}
                fi

```

## Deleting Releases

In addition to deleting tags, you might also want to delete the releases associated with these tags. You can do this using the GitHub CLI.

Here's a bash script that lists all releases, extracts the tag names, and then iterates over them. If a tag is greater than "v12.0", it deletes the corresponding release:

```bash
gh release list --json 'tagName' | jq -r '.[] | .tagName' | while read tag; do
    if [[ "$tag" > "v12.0" ]]; then
        echo "Deleting release $tag"
        gh release delete $tag --confirm
    fi
done
```

## Protected Tags

While this project demonstrates how to delete tags using GitHub Actions, a more secure and recommended approach is to use protected tags. 

Protected tags prevent contributors from creating or deleting tags that match certain patterns. This feature is available in public repositories with GitHub Free and GitHub Free for organizations, and in public and private repositories with GitHub Pro, GitHub Team, GitHub Enterprise Cloud, and GitHub Enterprise Server.

When you add a tag protection rule, all tags that match the pattern provided will be protected. Only users with admin or maintain permissions, or custom roles with the "edit repository rules" permission in the repository will be able to create protected tags, and only users with admin permissions or custom roles with the "edit repository rules" permission in the repository will be able to delete protected tags.

You can also import existing tag protection rules into repository rulesets. This will implement the same tag protections you currently have in place for your repository. Rulesets have several advantages over tag protection rules, such as multiple rulesets can apply at the same time, rulesets have statuses, anyone with read access to a repository can view the active rulesets for the repository, and with rulesets, you can restrict tag names on an organization-wide basis.

To add a tag protection rule or import tag protection rules to repository rulesets, you can navigate to the "Tags" section in the "Code and automation" section of the repository settings.

For more information, see the [GitHub documentation on tag protection rules](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/configuring-tag-protection-rules).