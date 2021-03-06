[![Python application test with Github Actions](https://github.com/acouprie/udacity-azure-project2/actions/workflows/main.yml/badge.svg)](https://github.com/acouprie/udacity-azure-project2/actions/workflows/main.yml)

# Overview of udacity-azure-project2

This project has been created during the [Azure DevOps Nanodegree on Udacity](https://www.udacity.com/course/cloud-devops-using-microsoft-azure-nanodegree--nd082).

After planning the different steps regarding the deployment of a Python Flask Machine Learning app, we will manage to add GitHub Actions and Azure Pipeline to it to have a fully working CI/CD environment.

## Project Plan

The planning tool I used are Trello, an Atlassian Todo list, and a Google Spreadsheets.

* [Trello dashboard](https://trello.com/b/q8sBlsrg/udacity)
* [Google Spreadsheets](https://docs.google.com/spreadsheets/d/1A4OkaFrMi0B3sxwRaSsHpANhVIFEv80CpgfdQOeLKO8/edit?usp=sharing)

Azure Cloud Shell

![Azure Cloud Shell](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/azure-cloud-shell.png)

Azure CI

![Azure CI](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/ci-diagram.png)

Azure CD

![Azure CD](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/cd-diagram.png)

# Instructions

## Set up Azure Cloud Shell

### Pair the SSH keys

Open the Azure shell and type:

```
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub
```

The id_rsa.pub file contains the key that needs to be copy and paste into GitHub.
(GitHub > Settings > SSH and GPG keys > Paste > Add the key).

Then you can clone the repository from the Azure Shell without typing your password.

```
git clone git@github.com:acouprie/udacity-azure-project2.git
```

![Azure project cloned](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/azure-ssh-clone.png)

### Create a virtual environment

```
python3 -m venv ~/.udacity-azure-project2
source ~/.udacity-azure-project2/bin/activate
```

### Install and run

```
make all
az webapp up -n udacity-azure-project2 -l eastus --sku B1
```

Your `make all` output should look like this:

![make all](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/make_all.png)

The `az webapp up ...` command will create a resource group that contains your App Service, something like this:

![Azure Portal](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/azure_portal.png)

/!\ In your case, `udacity-azure-project2` is likely taken, you have to put another name. Note also that with my subscription, I was only able to create one web app by location.
Following previous remark, modify the file `make_predict_azure_app.sh` line `-X POST https://udacity-azure-project2.azurewebsites.net:$PORT/predict`. You have to replace the `udacity-azure-project2` by the name of your application.

If you go to https://<your-adress>.azurewebsites.net, you should be able to see the following:

![Azure project running](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/application_running.png)

## Configure GitHub Actions

### Create the workflow

GitHub > Actions > set up a workflow yourself

Copy the following the replace the default template:

```
name: Python application test with Github Actions

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python 3.5
      uses: actions/setup-python@v1
      with:
        python-version: 3.5
    - name: Install dependencies
      run: |
        make install
    - name: Lint with pylint
      run: |
        make lint
    - name: Test with pytest
      run: |
        make test
```

After commiting, your build should be green. In details, it should look like this:

![GitHub Actions passed](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/github_actions_passed.png)

## Configure Azure Pipeline

Best way is to follow the official [Microsot documentation](https://docs.microsoft.com/en-us/azure/devops/pipelines/ecosystems/python-webapp?view=azure-devops).

![Azure Pipeline](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/azure_pipeline.png)

## Checking the application

The application is contacted from the `make_predict_azure_app.sh` file that return the prediction.

![Prediction](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/prediction.png)

### Access the logs

Use the following command to display the logs of the server:

```
az webapp log tail -g acouprie_rg_Linux_eastus -n udacity-azure-project2
```

Where -g is the resource group where your App Service is stored and -n is the actual App Service name.

![Logs](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/logs.png)

### Locust

Install locust with pip and run it:

```
pip install locust
locust
```

Go on http://0.0.0.0:8089/ and enter the URL of the App Service.

![Locust Main](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/locust-main.png)

After running:

![Locust Running](https://github.com/acouprie/udacity-azure-project2/blob/main/screenshots/locust-running.png)

## Enhancements

We should consider deploying with tools such as Docker or Kubernetes.
We should also consider having different environments related to different GitHub branches (staging and production) with Terraform and Packer templates to have a similare infrastructure between those environments. There could be a pipe between staging and production: if the code is correctly tested on staging, it can be deployed in production.

## Demo 

[Link to the video demo](https://youtu.be/hp1tQ1RR1xU)
