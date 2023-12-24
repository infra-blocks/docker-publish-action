# composite-action-template

Upon creating a repository from this template:
- Remove the [trigger-update-from-template workflow](.github/workflows/trigger-update-from-template.yml)
- Edit the action.yml to correspond to your new action
- Edit the self-test workflow.
- Edit this readme: this summary and the usage section.

## Usage

Describe usage of your action here.

## Development

### Releases

Releases of this action is automated by the CI. It is required to provide a semantic version bump as a label on the
PR. Upon merges, the CI will generate/update the appropriate tags. See [here](https://github.com/infrastructure-blocks/git-tag-semver-from-label-action) for more information.
