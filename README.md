# Add the github repo mirror into GCP

### Create a github token

Go to https://github.com/settings/tokens and `Generate token`

```
export GITHUB_TOKEN=WRITE-YOUR-TOKEN-HERE!!!
```

### Store gcloud access token

```
gcloud auth application-default login
export GCP_TOKEN=`gcloud auth application-default print-access-token`
export GCP_AUTH="Authorization: Bearer ${GCP_TOKEN}"
```

### Setup some ENVS
```
export GITHUB_AUTH="Authorization: token ${GITHUB_TOKEN}"
export GITHUB_USER=23inhouse
export GITHUB_REPO=website-23inhouse

export GCP_AUTH="Authorization: Bearer ${GCP_TOKEN}"
export GCP_PROJECT=fromatob-23inhouse
export GCP_PROJECT_ID=`gcloud projects describe ${GCP_PROJECT} --format='value(projectNumber)'`
export GCP_GITHUB_REPO=github-${GITHUB_USER}-${GITHUB_REPO}
export GCP_COMPUTE_ZONE=$(gcloud config get-value compute/zone)
export GCP_WEBHOOK_ID=FooBarBaz

export API_GH_KEYS=https://api.github.com/repos/${GITHUB_USER}/${GITHUB_REPO}/keys
export API_GH_HOOKS=https://api.github.com/repos/${GITHUB_USER}/${GITHUB_REPO}/hooks
export API_GCP_REPOS=https://sourcerepo.googleapis.com/v1/projects/${GCP_PROJECT}/repos
export API_GCP_TRIGGERS=https://cloudbuild.googleapis.com/v1/projects/${GCP_PROJECT}/triggers
```




### Create github mirroring webhook
```
curl -X POST ${API_GH_HOOKS} \
  -H "$GITHUB_AUTH" \
  -H "Content-Type: application/json" \
  -d @<(( echo "cat <<EOF" ; cat github-hook.json ; echo EOF ) | sh)

curl -H "${GITHUB_AUTH}" ${API_GH_HOOKS}

export GITHUB_HOOK=`curl -H "${GITHUB_AUTH}" ${API_GH_HOOKS} | jq 'reverse | .[0].id'`
echo ${GITHUB_HOOK}
```

### Create github ssh key
```
ssh-keygen -f ~/.ssh/gcp_id_rsa -t rsa -N ''
export GITHUB_SSHRSA=`cat ~/.ssh/gcp_id_rsa.pub`
echo ${GITHUB_SSHRSA}
```

### Create github deploy key
```
curl -X POST ${API_GH_KEYS} \
  -H "${GITHUB_AUTH}" \
  -H "Content-Type: application/json" \
  -d @<(( echo "cat <<EOF" ; cat github-key.json ; echo EOF ) | sh)

curl -H "${GITHUB_AUTH}" ${API_GH_KEYS}

export GITHUB_KEY=`curl -H "${GITHUB_AUTH}" ${API_GH_KEYS} | jq 'reverse | .[0].id'`
echo ${GITHUB_KEY}
```

### Enable source repos
```
gcloud services enable sourcerepo.googleapis.com
```

### Create the gcp mirrored repo
```
curl -X POST ${API_GCP_REPOS} \
  -H "${GCP_AUTH}" \
  -H "Content-Type: application/json" \
  -d @<(( echo "cat <<EOF" ; cat gcp-repo.json ; echo EOF ) | sh)

gcloud source repos list
gcloud source repos describe ${GCP_GITHUB_REPO} --format=json
```

### Enable cloudbuild
```
gcloud services enable cloudbuild.googleapis.com
```

### Create the gcp trigger
```
curl -X POST ${API_GCP_TRIGGERS} \
  -H "${GCP_AUTH}" \
  -H "Content-Type: application/json" \
  -d @<(( echo "cat <<EOF" ; cat gcp-trigger.json ; echo EOF ) | sh)

curl ${API_GCP_TRIGGERS} -H "${GCP_AUTH}" -H "Content-Type: application/json"
```


TODO: Where does `GCP_WEBHOOK_ID` in github-hook.json come from?

