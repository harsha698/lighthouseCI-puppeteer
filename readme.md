# Lighthouse CI with Puppeteer

## How to run?

### Prerequisites

- Node and npm

### Running Tests on local

1. Run `npm install`
2. Run `npm run audit`

### Run Tests on CI

## How does it work?

### How do we test authenticated URLs?

Lighthouse takes `puppeteerScript` as config. Via this script we simulate a user login journey. From here, Lighthouse takes over and audit all the pages whose urls are provided in `lighthouserc.js` file.

### Overall Flow
- In `npm run audit` script, we are triggering a command as `npx lhci autorun` which triggers `lighthouserc.js` file
- `lighthouserc.js` file hold all the values require for lighthouse ci to run.
- `setup.js` file is a puppeteer script which launches the browser, lhci will run the script for url1 for run1 then url2 for run1 and so on.
- For test purpose noOfRuns is set to 1.
- Counter is used in `setup.js` file so that login should happen once followed by auditing of all the urls.

### Report
- As lighthouse audit the pages, it generates reports for 4 metrics - performance, accessibility,  best-practices, seo and pwa and stores in `lhci_reports` folder.
- Report for every page gets generated in `.html` and `.json` formats.
- We can add or remove the metrics by using `lighthouserc.js` file

### Performance Metrics
- In `lighthouserc.js` file, assertions are mentioned indivisually for all the urls with some threshold for every metric.
- While auditing a page, if threshold for any of the metric is found to be greater than the actual value then test would fail.

## How to compare reports between runs?

### Prerequisites
- LHCI server should be up and running. It can be hosted on AWS or locally

### How to setup LHCI server locally?
- Start docker demon
- Can be done via heroku as well. However that requires heroku to be installed on machine. So used docker-compose.
- Create a docker-compose.yaml file inside lighthouse folder with below content:
```
version: '3'
services:
  lhserver:
    image: patrickhulce/lhci-server
    ports:
      - '9001:9001'
    volumes:
      - lhci-data:/data
volumes:
  lhci-data:
```
- Run command as `docker-compose up -d`
- Open browser and launch a url as `localhost:9001`. Ensure that no other process should run on this port. Else one should replace the port number in docker-compose file.
- Open terminal window.
- cd to this repo and folder lighthouse.
- Run below set of commands:
```
$ lhci wizard
? Which wizard do you want to run? new-project
? What is the URL of your LHCI server? http://localhost:9001/
? What would you like to name the project? lighthouse-ci-choicemax
Where is the project's code hosted? https://bitbucket.org/ideasorg/ui-performance/src/master/
```
- Output would be something like below:
```
Created project lighthouse-ci-choicemax (XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX)!
Use build token XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX to connect.
Use admin token XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX to manage the project.
```
- Open `lighthouserc.js` file.
- Replace upload object as below:
```
upload: {
            target: 'lhci',
            token: '<value-of-the-build-token-obtained-from-above-output>',
            ignoreDuplicateBuildFailure: true,
            serverBaseUrl:'http://localhost:9001/'
        },
```
- Run the tests