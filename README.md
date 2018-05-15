## Add the github repo mirror into GCP

gcloud source repos create github-website-23inhouse
git config credential.helper gcloud.sh
git remote add google https://source.developers.google.com/p/fromatob-23inhouse/r/github-website-23inhouse
git push google master
