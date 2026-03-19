
# Tray Workflow Sync

Automatically backup and version control your Tray workflows to GitHub using a native Tray workflow.

## Overview

This solution syncs Tray workflows from your Tray Organization to a GitHub repository for backup, version control, and compliance purposes.

## Features

- ✅ **Automatic sync** of workflow changes to GitHub
- ✅ **Visual workflow design** - easy to understand and modify
- ✅ **Two sync modes**: All workspaces or specific workspaces only
- ✅ **Smart change detection** - only syncs when workflows are modified
- ✅ **Clean file organization** by workspace, project name, and workflow name

## How It Works

1. **Fetches workflows** from your specified Tray workspaces
2. **Compares** with existing GitHub repository contents
3. **Categorizes changes** (new, updated, deleted workflows)
4. **Creates a branch** → **Pull Request** → **Merges changes**
5. **Cleans up** temporary branches

## Prerequisites

### GitHub Setup
1. **GitHub repository** where workflows will be stored
2. **GitHub auth** with repository write permissions
    a. Ensure that your auth has the `repo` scope selected when setting up in Tray.

### Tray Setup
1. **Tray account** with "contributor" access to workspace where solution exists
2. **Master token** for Tray APIs: https://tray.ai/documentation/tray-uac/governance/org-management/creating-api-tokens/

    **Note**: If you plan to sync multiple workspaces, you will need to change the permissons on your master token: https://www.loom.com/share/ba6d7c407fcc406d87c5070e1b62361f

## Installation

### Step 1: Create GitHub Repository

### Step 2: Import Tray Project template
The project contains several workflow templates that facilitate the sync process
1. Import the provided Tray project JSON into your Tray workspace
2. Set you authentications for github & Tray
3. Set the configuration variables:
    1. `branchName`: By default set to `automatic-updates`, can be anything you prefer or can be left as default. These branches are created, merged, and deleted automatically via the workflows.
    2. `git_repo`: This will be the name given in step 1.
    3. `git_user`: This will be the user or company account name where the repo exists, e.g.: `https://github.com/{git_user}/{git_repo}/tree/main`
    4. `syncAllWorkspaces`: defaults to `false`, which means you plan to specify which workspaces you wish to sync using the `specificWorkspaceIds` list. It's recommended that when you are first setting up the sync, you test with a specific set of workspace IDs. Then, if you wish to sync all workspaces, you can set this to `true`.
    5. `specificWorkspaceIds`: The specific workspace IDs you wish to sync if not syncing all workspaces. You can get this from the root workspace URL: `https://app.tray.io/workspaces/{workspaceId}/projects`

### Step 3: Test the Workflow
Ideally you decided to test with a workspace ID that has a limited number of projects/workflows in it. Open the `Sync step 1: Get workflows to sync` workflow and run the workflow. Depending on the number of workflows, this may take anywhere from a few seconds to a few minutes to run.

If you are running the sync for a large number of workflows, you will likely have some retries happening in the `Sync step 2: Process workflow change` workflow as many calls will be happening at once to github, this is ok usually, the the retries will normally resolve themselves after a few minutes.

This is because we are using a "threading architecture" for this solution, which means that we will make our calls to create / update all the files in parallel using "fire and forget" callable workflows.

This makes for a much faster initial sync for large installations. Note that for very large installations, you can adjust the inputs for the "Wait for threads" callable if you find that the solution is timing out. You could also introduce some intentional delays/backoffs if you find that the retries are timing out.

Once you have a fully successful run, check your github repo and make sure things are looking good.

### Step 4: Move solution to production
1. Select the workspaces you wish to sync: After you have tested things thoroughly, you can then make the final changes to sync all workspaces you wish to target. In this case, you could:
    1. Change the `syncAllWorkspaces` config option to `true`. This would sync all workspaces, including personal workspaces.
    2. Alternatively, you could add any additional workspace IDs to the `specificWorkspaceIds` config list of workspaces.
    3. Or, you could even create some sort of filtering logic after the "get all workspaces" graphQL call where you're filtering for certain types of workspaces only (e.g. "contains `prod` on the name" or "isn't a personal workspace")
    
    **Note**: changes to config variables sometimes take a few minutes to propogate. So if you flip the `syncAllWorkspaces` config option to `true` and run the workflow right away, that change may not take effect for the run and thus waiting a few minutes after making the change will be important.

2. Run a full sync - Now that your workspace strategy in place, run the `Sync step 1: Get workflows to sync` workflow so you can make sure that the full sync works as expected.
3. Schedule the sync
    A. Determine the frequency you wish to run the sync by switching the manual trigger on the `Sync step 1: Get workflows to sync` to a schedule trigger (e.g. nightly, weekly, etc)


## Configuration Options

### Sync Modes

#### Option A: Sync All Workspaces
```
syncAllWorkspaces: true
```
**Pros:**
- Automatically includes new workspaces
- Complete backup of all workflows
- Simpler setup

**Cons:**
- May include test/personal workspaces
- Larger repository size

#### Option B: Sync Specific Workspaces
```
syncAllWorkspaces: false,
specificWorkspaceIds: ['workspace-id-1', 'workspace-id-2']
```
**Pros:**
- Precise control over synced content
- Exclude test/development workspaces
- Better performance

**Cons:**
- Manual updates needed for new workspaces
- More complex configuration

### Finding Workspace IDs
1. Go to your Tray workspace
2. Check the URL: `https://app.tray.io/workspaces/{workspace-id}/...`
3. Or use the Tray GraphQL explorer to query workspace information

## Scheduling
Set up automatic execution:
1. **Trigger Type**: Time-based
2. **Recommended Schedule**: 
   - Hourly for active development
   - Daily for production environments
3. **Timezone**: Set to your team's timezone

## File Structure

Workflows are organized in GitHub as:

```
Repository/
├── Workspace Name/
│   └── Project Name/
│       ├── Workflow Name 1/
│       │   └── workflow-id-123.json
│       └── Workflow Name 2/
│           └── workflow-id-456.json
└── Another Workspace/
    └── Different Project/
        └── Different Workflow/
            └── workflow-id-789.json
```

**Note**: Forward slashes in workspace/workflow names are replaced with `／` to avoid directory conflicts.

---

**Version**: 1.0  
**Last Updated**: [Current Date]  
**Compatible with**: Tray.io Platform