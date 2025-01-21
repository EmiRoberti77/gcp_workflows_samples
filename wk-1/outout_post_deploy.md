## output post deploy

```bash
./deploy.sh
                                                                                              1
Waiting for operation [operation-1737444219513-62c3241856c56-5765c87d-73fd8fe4] to complete...done.
createTime: '2025-01-21T07:23:39.652414148Z'
name: projects/gketest-448214/locations/us-west1/workflows/emi_wf_sample_1
revisionCreateTime: '2025-01-21T07:23:39.678880900Z'
revisionId: 000001-cff
serviceAccount: projects/gketest-448214/serviceAccounts/51152465653-compute@developer.gserviceaccount.com
sourceContents: |
  main:
    steps:
      - logStart:
          call: sys.log
          args:
            text: 'Workflow started'
      - fetchWeather:
          call: http.get
          args:
            url: 'https://api.open-meteo.com/v1/forecast?latitude=35&longitude=139&hourly=temperature_2m'
          result: weatherResponse
      - logResponse:
          call: sys.log
          args:
            text: ${"Weather API response:" + weatherResponse.body}
      - completed:
          return: 'Workflow finished'
state: ACTIVE
updateTime: '2025-01-21T07:23:39.956254212Z'
```
