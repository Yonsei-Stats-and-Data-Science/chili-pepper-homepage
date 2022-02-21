<br/>
<p align="center">
    <a href="https://hpc.stat.yonsei.ac.kr/" target="_blank">
        <img width="50%" src="https://hpc.stat.yonsei.ac.kr/images/logo.svg" alt="Yonsei BK logo">
    </a>
</p>

<br/>
<p align="center">
</p>
<br/>

<p align="center">
    <a href="https://hpc.stat.yonsei.ac.kr">
        <img width="20%" src="https://hpc.stat.yonsei.ac.kr/images/chili.png" alt="Chili Pepper Cluster Logo">
    </a>
</p>
<br/>

# [Chili Pepper Landing Page](https://hpc.stat.yonsei.ac.kr)

Static site landing page built with Hugo.

Use webserver(Nginx) to host the `deploy` branch of `landing-page` repository. The compilation of static files are done remotely via GitHub Actions.

Go to the `/mnt/nas/public` directory and download the `deploy` branch likewise.

```bash
cd /mnt/nas/public
git clone git@github.com:Yonsei-Stats-and-Data-Science/landing-page.git -b deploy
```


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
