# tf-plan-apply-buildkite-plugin

This plugin can be used to run `terraform`. It is useful in scenarios where you want to run a `plan` on branches which are not your `main`/`master` branch and control the `apply` on those branches by firstly running a `plan` which you can agree to run via a `block` step.

Table of contents:
- [tf-plan-apply-buildkite-plugin](#tf-plan-apply-buildkite-plugin)
  - [Getting Started](#getting-started)
    - [Example: Plan Only](#example-plan-only)
    - [Example: Apply via block step](#example-apply-via-block-step)
  - [Configuration](#configuration)
  - [Contributing](#contributing)
  - [Versioning](#versioning)
  - [Authors](#authors)
  - [License](#license)

## Getting Started

Please see the below examples on how to use this plugin with buildkite. The [buildkite-agent](https://buildkite.com/docs/agent/v3) also requires access to `docker`.

### Example: Plan Only

```yaml
steps:
  - label: ":terraform: Plan only"
    if: "build.branch != main && build.branch != master"
    plugins:
      - ssh://git@github.com/jack1902/tf-plan-apply-buildkite-plugin.git#v0.0.1:
          action: plan
          version: 0.14.5
    agents:
      queue: default
```

This runs a `terraform plan` and all output is visible in the buildkite log. For additional options that you can pass, see [Configuration](#configuration)


### Example: Apply via block step

```yaml
steps:
  - label: ":terraform: Terraform apply via block step"
    branches: main master
    plugins:
      - ssh://git@github.com/jack1902/tf-plan-apply-buildkite-plugin.git#v0.0.1:
          action: block_apply
          version: 0.14.5
    agents:
      queue: default
```

This runs the `terraform plan` with the `-detailed-exitcode` flag which is used to detect changes.

Depending on the exitcode given by `terraform` the following things happen:

- `No Changes` - Annotate the pipeline stating that there are `No Changes` for the given `Directory + Workspace`
- `Changes Detected` - Annotate the pipeline stating there are `Changes` , and upload a `Block & Apply` . This enables YOU to state `unblock` to the given `plan` and know what changes are going to happen without just blindly running `terraform apply -auto-approve`
- `Error` - Pipeline blows up as `terraform` encountered errors


## Configuration

| Option              | Required |   Type    |        Default        | Description                                                                                                                |
| ------------------- | :------: | :-------: | :-------------------: | -------------------------------------------------------------------------------------------------------------------------- |
| version             |   Yes    | `string`  |       `0.14.5`        | The Terraform version or Image Tag to use to run Terraform                                                                 |
| action              |   Yes    | `string`  |                       | What action to take. Should be one of `plan` or `block_apply`                                                              |
| debug               |    No    | `boolean` |        `false`        | Run the plugin with verbose logging                                                                                        |
| workspace           |    No    | `string`  |       `default`       | The `terraform workspace` to against                                                                                       |
| terraform_directory |    No    | `string`  |          `.`          | The Working Directory to run within (helpful for a mono-repo of terraform)                                                 |
| var_file            |    No    | `string`  |                       | `-var-file` to pass to `terraform plan` / `terraform apply`                                                                |
| image_name          |    No    | `string`  | `hashicorp/terraform` | The image_name to use to run `terraform` (Helpful to change if you need `helm` and/or `kubectl` within a custom container) |
| validate            |    No    | `boolean` |        `true`         | Whether or not to run `terraform validate`                                                                                 |

## Contributing

Feel free to open `issues` and open Pull Requests to fix any bugs or extend the feature set of this plugin for your `terraform` use-case.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/jack1902/tf-plan-apply-buildkite-plugin/tags).

## Authors

* **Jack** - *Initial work* - [jack1902](https://github.com/jack1902)

See also the list of [contributors](https://github.com/jack1902/tf-plan-apply-buildkite-plugin/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details
