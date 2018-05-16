# Add the github repo mirror into GCP

### Setup some ENVS
```
export GITHUB_USER=23inhouse
export GITHUB_REPO=website-23inhouse

export GCP_PROJECT=fromatob-23inhouse
export GCP_PROJECT_ID=`gcloud projects describe ${GCP_PROJECT} --format='value(projectNumber)'`
export GCP_GITHUB_REPO=github-${GITHUB_USER}-${GITHUB_REPO}
export GCP_COMPUTE_ZONE=$(gcloud config get-value compute/zone)
export GCP_AUTH=`gcloud auth application-default print-access-token`
```

### Create a github token

Go to https://github.com/settings/tokens and `Generate token`

```
export GITHUB_AUTH="Authorization: token YOUR-TOKEN"
curl -H "$GITHUB_AUTH" https://api.github.com/repos/${GITHUB_USER}/${GITHUB_REPO}/keys
```

### Create mirroring webhook
```
curl -X POST https://api.github.com/repos/${GITHUB_USER}/${GITHUB_REPO}/hooks \
  -H "$GITHUB_AUTH" \
  -d @<(( echo "cat <<EOF" ; cat github-hook.json ; echo EOF ) | sh)

curl -H "${GITHUB_AUTH}" https://api.github.com/repos/${GITHUB_USER}/${GITHUB_REPO}/hooks

export GITHUB_HOOK=`curl -H "${GITHUB_AUTH}" https://api.github.com/repos/${GITHUB_USER}/${GITHUB_REPO}/hooks | jq 'reverse | .[0].id'`
```

### Create deploy key
```
curl -X POST https://api.github.com/repos/${GITHUB_USER}/${GITHUB_REPO}/keys \
  -H "${GITHUB_AUTH}" \
  -d @<(( echo "cat <<EOF" ; cat github-key.json ; echo EOF ) | sh)

curl -H "${GITHUB_AUTH}" https://api.github.com/repos/${GITHUB_USER}/${GITHUB_REPO}/keys

export GITHUB_KEY=`curl -H "${GITHUB_AUTH}" https://api.github.com/repos/${GITHUB_USER}/${GITHUB_REPO}/keys | jq 'reverse | .[0].id'`
```

### Create the gcp mirrored repo
```
curl -X POST https://sourcerepo.googleapis.com/v1/${GCP_PROJECT}/repos \
  -H "Content-Type: application/json" \
  -d @<(( echo "cat <<EOF" ; cat @gcp-repo.json ; echo EOF ) | sh)

gcloud source repos list
gcloud source repos describe ${GCP_GITHUB_REPO} --format=json
```


### Create the gcp trigger
```
curl -X POST https://cloudbuild.googleapis.com/v1/projects/${GCP_PROJECT_ID}/triggers \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${GCP_AUTH}" \
  -d @<(( echo "cat <<EOF" ; cat @gcp-trigger.json ; echo EOF ) | sh)

curl -X GET https://cloudbuild.googleapis.com/v1/projects/${GCP_PROJECT_ID}/triggers \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${GCP_AUTH}"
```


TODO: Where does the `ssh-rsa` `key` in github-key.json come from?
TODO: Where does `PNnueMrfOA` in github-hook.json come from?

