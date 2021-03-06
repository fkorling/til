# Use labels on Cloud Run services for a billing breakdown

Thanks to [@glasnt](https://github.com/glasnt) for the tip on this one. If you want a per-service breakdown of pricing on your Google Cloud Run services within a project (each service is a different deployed application) the easiest way to do it is to apply labels to those services, then request a by-label pricing breakdown.

This command will update a service (restarting it) with a new label:

```bash
gcloud run services update csvconf --region=us-central1 --platform=managed --update-labels service=csvconf
```

I found it needed the `--platform=managed` and `--region=X` options to avoid it asking interactive questions.

I applied this to all of my services by runnning this command:

```
gcloud run services list --platform=managed | tail -n +2
+  asgi-log-demo                  us-central1  https://asgi-log-demo-j7hipcg4aq-uc.a.run.app                  email@gmail.com                                           2020-04-22T00:40:39.483Z
+  cloud-run-hello                us-central1  https://cloud-run-hello-j7hipcg4aq-uc.a.run.app                email@gmail.com                                           2020-04-22T00:46:48.251Z
+  covid-19                       us-central1  https://covid-19-j7hipcg4aq-uc.a.run.app                       email@gmail.com                                           
```

I tried figuring out how to use awk to get at the different fields, then gave up and copied them all into Visual Studio Code and used multi-line editing to turn them into:

```
gcloud run services update asgi-log-demo --region=us-central1 --platform=managed --update-labels service=asgi-log-demo
gcloud run services update cloud-run-hello --region=us-central1 --platform=managed --update-labels service=cloud-run-hello
gcloud run services update covid-19 --region=us-central1 --platform=managed --update-labels service=covid-19
```

I saved that as a `runme.sh` script and used `. runme.sh` to run it.

The output of the script looked like this (one entry for each service) - each one took ~30s to run.
```
Service [covid-19] revision [covid-19-00122-zod] has been deployed and is serving 100 percent of traffic at https://covid-19-j7hipcg4aq-uc.a.run.app
✓ Deploying... Done.                                                                                                                                                                                                 
  ✓ Creating Revision...                                                                                                                                                                                             
  ✓ Routing traffic...                                                                                                                                                                                               
Done.                                                                                                                                                                                                                
```

I think it takes a while for the labelled billing breakdown to start showing up. If I'm right then https://console.cloud.google.com/billing/00C25B-639CE5-5833F9/reports;chartType=STACKED_BAR;grouping=GROUP_BY_LABEL_KEYS;groupingLabel=serving.knative.dev%2Fservice;products=services%2F152E-C115-5142 will show me this breakdown at some point, maybe tomorrow? I'll add a screenshot when it does.
