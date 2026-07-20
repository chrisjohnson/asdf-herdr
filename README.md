<div align="center">

# asdf-herdr [![Build](https://github.com/chrisjohnson/asdf-herdr/actions/workflows/build.yml/badge.svg)](https://github.com/chrisjohnson/asdf-herdr/actions/workflows/build.yml) [![Lint](https://github.com/chrisjohnson/asdf-herdr/actions/workflows/lint.yml/badge.svg)](https://github.com/chrisjohnson/asdf-herdr/actions/workflows/lint.yml)

[herdr](https://herdr.dev) plugin for the [asdf version manager](https://asdf-vm.com).

</div>

# Contents

- [Dependencies](#dependencies)
- [Install](#install)
- [Contributing](#contributing)
- [License](#license)

# Dependencies

- `bash`, `curl`, `git`, and [POSIX utilities](https://pubs.opengroup.org/onlinepubs/9699919799/idx/utilities.html).
- Prebuilt binaries are fetched for macOS and Linux, on `x86_64` and `arm64`.

# Install

Plugin:

```shell
asdf plugin add herdr
# or
asdf plugin add herdr https://github.com/chrisjohnson/asdf-herdr.git
```

herdr:

```shell
# Show all installable versions
asdf list-all herdr

# Install specific version
asdf install herdr latest

# Set a version globally (on your ~/.tool-versions file)
asdf global herdr latest

# Now herdr commands are available
herdr --version
```

Check [asdf](https://github.com/asdf-vm/asdf) readme for more instructions on how to
install & manage versions.

# Contributing

Contributions of any kind welcome!

# License

See [LICENSE](LICENSE) © [Chris Johnson](https://github.com/chrisjohnson/)
