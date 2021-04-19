# OpenShift Compliance Operator

1. Install Operator - Use OperatorHub and search for `Compliance Operator` and install it.  Wait a few minutes.

2. Use correct project.  All instructions from here assume you are using this project.

```
oc project openshift-compliance
```

3. See installed Profiles

```
oc get profiles.compliance
```

4. View Details

```
oc get -oyaml profiles.compliance <profile name>
# ocp4-cis for instance
```

5. View Rules
```
oc get -oyaml rules.compliance <rule_name>
# ocp4-accounts-restrict-service-account-tokens for instance
```

6.  Create `ScanSetting`

```
# Web Console
Name: jon-scan-setting
Label: app=jon-compliance
```

or

```
# CLI
oc apply -f ./0_scan_settings.yml
```

7.  Create `ScanSettingBinding`

```
# Web Console
Name: nist-moderate
Label: app=jon-compliance
```

or

```
# CLI
oc apply -f ./1_scan_settings_binding.yml
```

8.  Verify

```
# takes a few minutes to complete
# watch compliance suite to see progress of scan
oc get compliancesuites -w

# watch individual pods
oc get pods -w
```

9.  Check Results

```
# to see results of scan
oc get compliancecheckresult

# to check remediations
oc get complianceremediations

# to actually apply remediation, and look for apply: false and change that to apply: true
oc edit complianceremediation/rhcos4-moderate-worker-no-direct-root-logins
# or oc edit complianceremediation/<whatever rule>
```

10.  See Volumes

```
# see pv
oc get pv

# see claims. once done, will see one name pvc/workers-scan and one for pvc/masters-scan
oc get pvc
```

11. Deploy Pod That Mounts the Volume

```
# create pod, give it a minute
oc apply -f ./2_pv_extract_pod.yml

# see scan results in volume
oc exec pods/pv-extract -- ls /workers-scan-results/0
```

12. Copy results locally

```
oc cp pv-extract:/workers-scan-results .
```

13. Unzip

```
cd 0
bunzip ./openscap-pod-c9cf5cdf58bbffcb811a8f7666c3d463c590d70f.xml.bzip2
# look for *.xml.out file
```

14. Delete Pod

```
# if you want
oc delete pod pv-extract
```



## References
1.  [Github](https://github.com/openshift/compliance-operator)
2.  [OpenShift Docs](https://docs.openshift.com/container-platform/4.6/security/compliance_operator/compliance-operator-understanding.html)
3.  [OpenShift Troubleshooting Docs](https://docs.openshift.com/container-platform/4.6/security/compliance_operator/compliance-operator-troubleshooting.html)
4.  [Great Demo on Youtube](https://www.youtube.com/watch?v=phillosgRyo)
