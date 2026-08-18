<div align="center">

# asdf-herdr [![Build](https://github.com/chrisjohnson/asdf-herdr/actions/workflows/build.yml/badge.svg)](https://github.com/chrisjohnson/asdf-herdr/actions/workflows/build.yml) [![Lint](https://github.com/chrisjohnson/asdf-herdr/actions/workflows/lint.yml/badge.svg)](https://github.com/chrisjohnson/asdf-herdr/actions/workflows/lint.yml)

[herdr](https://herdr.dev) plugin for the [asdf version manager](https://asdf-vm.com).

Compatible with asdf 0.17.0+.

</div>

# Contents

- [Dependencies](#dependencies)
- [Install](#install)
- [Usage](#usage)
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

# Usage

```shell
# Show installed versions
asdf list herdr

# Show current version
asdf current herdr

# Switch to a different version
asdf local herdr <version>

# Uninstall a version
asdf uninstall herdr <version>
```

# Contributing

Contributions of any kind welcome!

## Development

1. Clone this repository
2. Make changes
3. Test with:
   ```shell
   asdf plugin remove herdr
   asdf plugin add herdr /path/to/cloned/repo
   asdf install herdr latest
   herdr --version
   ```

# License

See [LICENSE](LICENSE) © [Chris Johnson](https://github.com/chrisjohnson/)
