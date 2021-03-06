# Grinder Executor

## Running Grinder

Couple of basic example configs can be found [here](https://github.com/Blazemeter/taurus/tree/master/examples/grinder). 

Running Grinder with existing script and properties file:
```yaml
execution:
- executor: grinder
  concurrency: 3
  ramp-up: 10s
  iterations: 20
  scenario: script_sample
  
scenarios:
  script_sample:
    script: tests/grinder/helloworld.py
    properties-file: tests/grinder/grinder.properties
    properties:
      grinder.useConsole: false
```

Generating Grinder script from scenario:
```yaml
execution:
- executor: grinder
  concurrency: 20
  ramp-up: 1m
  iterations: 100
  scenario: requests_sample

scenarios:
  requests_sample:
    default-address: http://blazedemo.com  # base address for requests
    think-time: 1s                         # pause for 1s after each request
    timeout: 30s                           # specify timeout for requests
    headers:                               # global headers, applied to all requests
      X-Api-Key: my-fresh-token
    store-cookie: false                    # simulate browser cookie storage (default value is `true`)
    keepalive: true                        # flag to use keep-alive for connections, default is `true`  
    requests:
    - /                                    # short form, URL only
    - url: /reserve.php                    # full form
      method: POST
      think-time: 5s
    - url: /payment.php
      method: POST
      headers:                             # request-specific headers
        Content-Type: application/json
      think-time: 1s                       # overrides scenario-level `think-time`
```

## Notes on Load Settings

Grinder module of Taurus supports following taurus execution parameters: `concurrency`, `ramp-up`, `hold-for` and `iterations` (all except of `throughput`). When you configure limit of `iterations`, it applies to every worker thread so overall script iteration count can be up to `concurrency` * `iterations`. All parameters are passed with .properties file, so they will work for your pre-existing script too (not for generated scripts only).
 
By default, only one worker process of grinder will be spawned. You can change that by using `properties` section of scenario config. 

For your internal scripting needs, Taurus writes all load settings from execution section into grinder properties, with following names:

 - `taurus.concurrency`
 - `taurus.throughput` 
 - `taurus.ramp\_up`
 - `taurus.steps`
 - `taurus.hold\_for`
 - `taurus.iterations`

So, you can access values of those properties from your Jython test script like this: `grinder.properties.getPropertySubset('taurus.').getInt('concurrency', 1)`

For example, you might want to use `taurus.ramp\_up` value to simulate thread ramp up like Grinder docs [suggest](http://grinder.sourceforge.net/g3/script-gallery.html#threadrampup.py). If you will look into .py file inside artifacts dir, generated by YAML scenario, you'll see how exactly it's done. 

## Module settings

```yaml
modules:
  grinder:
    path: ~/.bzt/grinder-taurus/ # Path to Grinder. If no grinder.jar found in folder/lib/, Grinder tool will be automatically downloaded and installed in "path". 
    download-link: http://sourceforge.net/projects/grinder/files/The%20Grinder%203/{version}/grinder-{version}-binary.zip/download  # Link to download Grinder from
    report-by-url: false  # change results analysis to use URLs instead of test ID/test name
```