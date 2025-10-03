<div align="center">

  <!-- Row of icons (optional, uncomment if needed) -->

  <p>
    <img src="https://logo.svgcdn.com/d/ansible-original-wordmark.svg" alt="Ansible" height="95" />
  </p>


  <h1>Ansible Production Grade Setup</h1>

  <p>
    Production-grade Ansible playbooks and roles for infrastructure automation and configuration management.
  </p>

  <p>
    <a href="https://github.com/sean-njela/ansible-demo/graphs/contributors">
      <img src="https://img.shields.io/github/contributors/sean-njela/ansible-demo" alt="contributors" />
    </a>
    <a href="">
      <img src="https://img.shields.io/github/last-commit/sean-njela/ansible-demo" alt="last update" />
    </a>
    <a href="https://github.com/sean-njela/ansible-demo/network/members">
      <img src="https://img.shields.io/github/forks/sean-njela/ansible-demo" alt="forks" />
    </a>
    <a href="https://github.com/sean-njela/ansible-demo/stargazers">
      <img src="https://img.shields.io/github/stars/sean-njela/ansible-demo" alt="stars" />
    </a>
    <a href="https://github.com/sean-njela/ansible-demo/issues/">
      <img src="https://img.shields.io/github/issues/sean-njela/ansible-demo" alt="open issues" />
    </a>
    <a href="https://github.com/sean-njela/ansible-demo/blob/master/LICENSE">
      <img src="https://img.shields.io/github/license/sean-njela/ansible-demo.svg" alt="license" />
    </a>
  </p>
</div>

## Screenshots

<details>
  <summary>Click to expand</summary>
  <!-- Example (uncomment if needed) -->
  <div align="center">
    <img src="assets/screenshot1.png" alt="screenshot1" width="800" />
    <img src="assets/screenshot2.png" alt="screenshot2" width="800" />
  </div>
</details>

## Tech Stack

> List of tools used in the project

[![Devbox](https://www.jetify.com/img/devbox/shield_moon.svg)](https://www.jetify.com/devbox/docs/contributor-quickstart/)
![Taskfile](https://img.shields.io/badge/Taskfile-3.44.0-green)
![gitflow](https://img.shields.io/badge/gitflow-1.12-green)
![uv](https://img.shields.io/badge/uv-0.8-green)
![precommit](https://img.shields.io/badge/precommit-4.3.0-green)

## Prerequisites

> [!IMPORTANT]
> This project uses **Devbox** to provide a consistent development environment.

1. **Install Docker**
   [Docker installation guide](https://docs.docker.com/get-docker/)

2. **Install Devbox**
   [Devbox installation guide](https://www.jetify.com/devbox/docs/installing_devbox/)

3. **Clone the repository**
   ```bash
   git clone ...
   cd ...
   ```

4. **Start Devbox shell**

   ```bash
   devbox shell
   ```

  > First run may take several minutes to install tools, but subsequent runs spin up in seconds.

## Quick Start

```bash
task setup
task status   # check if everything is running
task dev      # start development stack
task info     # to list urls to visit
task cleanup-dev
```

## Documentation

For full documentation, setup instructions, and architecture details, visit the [docs](docs/index.md) directory or run locally with:

```bash
task docs
```

Then open: [http://127.0.0.1:8030/]()

## Tasks (Automation)

> [!IMPORTANT]
> This project is designed for a simple, one-command setup. All necessary actions are orchestrated through `Taskfile.yml`.

The `Taskfile.gitflow.yml` provides a structured Git workflow using Git Flow. This helps in managing features, releases, and hotfixes in a standardized way. To run these tasks just its the same as running any other task. Using gitflow is optional. If you do not want the gitflow tasks, you can remove the `Taskfile.gitflow.yml` file and unlink it from the `Taskfile.yml` file (remove the `includes` section). If you cannot find the section use CTRL + F to search for `Taskfile.gitflow.yml`.

To see all tasks:

```bash
task --list-all
```

## Contributing

<a href="https://github.com/sean-njela/ansible-demo/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=sean-njela/ansible-demo" />
</a>

> Contributions welcome! Open an issue or submit a PR.

## License

Distributed under the MIT License. See `LICENSE` for more info.

## Contact

* [LinkedIn](https://linkedin.com/in/sean-njela)
* [Twitter/X](https://x.com/devopssean)
* [seannjela@outlook.com](mailto:seannjela@outlook.com)
* [About Me](docs/4-about/0-about.md)
