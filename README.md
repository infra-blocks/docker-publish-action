# docker-publish-action

This action, contrary to its name, does more than publish a docker image. It builds, tags and pushes images to
repositories.

It's biased towards using `docker compose` for static build definition, such as context, image repository,
dockerfile location, environment variables, build arguments, etc... Therefore, instead of passing all those as inputs
to the action, the user passes a `docker-compose.yml` file.

The action will leverage `docker compose` to build the image. Then, it will tag it with the tags the users provided
as inputs.

The process ends by pushing the tagged images to the remote repository.

NOTES:
- This action does not log into a docker registry. This is expected to be done outside and before calling this
action.
- This action can run on any event and it is made to be embedded into a more turnkey workflow, where the tags
are intelligently determined depending on the context of the invocation.

## Inputs

|        Name         | Required | Description                                                                             |
|:-------------------:|:--------:|-----------------------------------------------------------------------------------------|
|       service       |   true   | The name of the service to build as found in the docker-compose file.                   |
|        tags         |  false   | A stringified JSON array of tags to apply on the built image. Defaults to '["latest"]'. |
| docker-compose-file |  false   | The location of the docker-compose file. Defaults to "docker/docker-compose.yml".       |
|       dry-run       |  false   | When set to true, builds and tags, but doesn't push images.                             |

## Outputs

|   Name    | Description                                                                           |
|:---------:|---------------------------------------------------------------------------------------|
| published | A stringified JSON array of images published. An image is of the form '<repo>:<tag>'. |

## Permissions

N/A

## Usage

The following example shows an invocation on an ECR public repository, using GitHub OIDC provider to authenticate
with AWS.

```yaml
jobs:
  build-docker-images:
    runs-on: ubuntu-22.04
    permissions:
      id-token: write
      contents: read
    env:
      AWS_REGION: us-east-1
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ vars.AWS_ROLE }}
          aws-region: ${{ env.AWS_REGION }}
      - uses: aws-actions/amazon-ecr-login@v2
        with:
          registry-type: public
      - name: Build, tag and publish docker image
        id: docker-publish
        uses: infrastructure-blocks/docker-publish-action@v1
        with:
          service: action
          tags: '["big-tag", "bigger-tag"]'
      - name: Show published images
        run: |
          echo "${{ steps.docker-publish.outputs.published }}"
```

The above example implies a `docker/docker-compose.yml` file that could look like this:
```yaml
services:
  my-image:
    image: public.ecr.aws/<your-alias>/<your-image>
    build:
      context: ../
      dockerfile: docker/action/Dockerfile
    container_name: my-image
```

## Development

### Releasing

The releasing is handled at git level with semantic versioning tags. Those are automatically generated and managed
by the [git-tag-semver-from-label-workflow](https://github.com/infrastructure-blocks/git-tag-semver-from-label-workflow).
