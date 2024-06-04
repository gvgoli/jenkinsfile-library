# jenkinsfile-library
CloudBees Jenkinsfile Library

# Repo Requirements:
- Webhook:
  - GitHub steps in repo:
    - Settings -> Hooks
      - Payload URL: https://cloudbees.agtservices-nonprod.aegon.io/##CONTROLLER-NAME###/github-webhook/
      - Content type: application/json
      - SSL verification: Enable SSL verification
      - Which events would you like to trigger this webhook?:  Let me select individual events.
        - Check:
          - Branch or tag creation
          - Branch or tag deletion
          - Pull requests
          - Pushes
          - Active
- ci.yaml file - info below

## ci.yaml example
```
Build:
- pip install -r src/requirements-dev.txt
Test:
- python src/tests/conftest.py
- python src/tests/test_common.py
- python src/tests/test_incident.py
Snyk:
  org_slug: aegon-gts
  package_manager: pip
  target_file: src/requirements-dev.txt
Artifact: false
```
-   #### Build:
    - commands to build application
-   #### Test:
    - commands to test application
-   #### Snyk:
    - Organization Slugs:
      - Aegon GTS = aegon-gts
      - TA        = aegon-template
      - AAM       = aegon-aam
    - Supported package managers: https://docs.snyk.io/getting-started/supported-languages-and-frameworks
        - rubygems (RubyGems),
        - npm (npm),
        - yarn (Yarn),
        - maven (Maven),
        - pip (pip),
        - sbt (SBT),
        - gradle (Gradle),
        - golangdep (dep (Go)),
        - gomodules (Go Modules),
        - govendor (govendor),
        - nuget (NuGet),
        - paket (Paket),
        - composer (Composer),
        - cocoapods (CocoaPods),
        - poetry (Poetry),
        - hex (Hex),
        - Unmanaged (C/C++) (Unmanaged (C/C++)),
        - swift (Swift)
    - target_file: same file for building app.
-   #### Nexus Repository: Only set when not wanting to create an Artifact. Default = True
    - Artifact: false  