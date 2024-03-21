# GSP897
## Run in cloudshell
```cmd
export PROJECT_ID=$(gcloud config get-value core/project)
gsutil mb gs://${PROJECT_ID}
gsutil -m cp gs://cloud-training/gsp897/cosmetic-test-data/*.png \
gs://${PROJECT_ID}/cosmetic-test-data/
gsutil ls gs://${PROJECT_ID}/cosmetic-test-data/*.png > /tmp/demo_cosmetic_images.csv
gsutil cp /tmp/demo_cosmetic_images.csv gs://${PROJECT_ID}/demo_cosmetic_images.csv
```
## Steps :-
### Go to `Navigation Menu` > `Visual Inspection AI`
### Click the `Enable Visual Inspection AI API` button
### Click `Create a Dataset`
### For Dataset name, enter `cosmetic`
### For Objective select `Cosmetic Inspection`.
### For Annotation type select `Polygon`.
### For Region, select `US Central1`.
### Click Create.
### On the Import tab, under Select an import method, select Select an import file from Cloud Storage.
### In Import File Path, click Browse.
### Expand the bucket with a name that matches the lab project ID.
### Select the demo_cosmetic_images.csv import file.
### Click Select.
### Click Continue.
