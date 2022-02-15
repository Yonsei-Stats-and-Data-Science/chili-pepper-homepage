# landing-page

Static site landing page built with Hugo.

Use webserver(Nginx) to host `./public`. 

## Installation

1. Install `go` https://go.dev/doc/install.

2. Install extended version of `hugo` https://gohugo.io/getting-started/installing/.

3. Clone from repository with `--recursive` flag to pull theme.

```bash
git clone --recursive git@github.com:Yonsei-Stats-and-Data-Science/landing-page.git
```

## Usage

### Compiling static sites

The following command will create the static site based on `config.yml` and files in subdirectories.

```bash
hugo
```

Upon updates, static sites must be newly built using the above command. It is recommended to erase the `./public` folder for a proper updated build.

### Creating Documentation

The following command will create a templated markdown file in `./content/docs/new-content.md`.

```bash
hugo new docs/new-content.md
```

### Live localhost server

For debugging purposes the live preview for the hugo website can be accessed on `localhost:1313/` with the following command. 

```bash
hugo server -D
````
