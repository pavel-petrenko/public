Example how to Update <br/>Microsoft Azure Attestation Policy
===

```js
//
// Example how to update Microsoft Azure Attestation Policy
//
// Steps: 
//   1: Get JIT for prod subscription.
//   2: Login to SAW using your AME account 
//   3: Navigate in the Browser to an Attestation Provider and Copy "Attest URL" to this script
//   4: Open Browser Dev Console -> Network Connections Tab
//   5: Navigate to "Policy" blade (tab in Attestation Provider UI)
//   6: Clear Network Log (press Ctrl+L)
//   7: Choose Attestation Type that must be updated and that generate new records in Network Log (must be just some ~ 5 or 7 lines)
//   8: Choose record with GET request for the current policy and copy Authorization Token from the log to this script.
//   9: Update policy in this script (just policy text. do not change the code)
//   10: Now at your DEV box open http://portal.azure.com/
//   11: Open Browser Dev Tools there and Navigate to "Console Tab"
//   12: Put in console this whole script and execute. As result you'll get response with updated policy. 
//   Enjoy.
//

const attestURL = 'https://cus.cus.attest.azure.net/';
const attestationType = 'VbsEnclave';

const encodedPolicyBody = btoa(`
  version= 1.1;
  configurationrules{
    => issueproperty(type="x-ms-sgx-tcbidentifier", value="10");
  };
  authorizationrules{
    => permit();
  };
  issuancerules{
    c:[type=="x-ms-sgx-is-debuggable"] => issue(type="is-debuggable", value=c.value);
    c:[type=="x-ms-sgx-mrsigner"]      => issue(type="sgx-mrsigner", value=c.value);
    c:[type=="x-ms-sgx-mrenclave"]     => issue(type="sgx-mrenclave", value=c.value);
    c:[type=="x-ms-sgx-product-id"]    => issue(type="product-id", value=c.value);
    c:[type=="x-ms-sgx-svn"]           => issue(type="svn", value=c.value);
    c:[type=="x-ms-attestation-type"]  => issue(type="tee", value=c.value);
  };
`).replace(/=*$/g, '').replace(/\+/g, '-').replace(/\//g, '_');

const encodedJWTBody = btoa('{ "AttestationPolicy": "' + encodedPolicyBody + '" }').replace(/=*$/g, '').replace(/\+/g, '-').replace(/\//g, '_');

const encodedPolicy = 'eyJhbGciOiJub25lIn0.' + encodedJWTBody + '.';

const token = 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJ..........................................................qWFQfw';

// Get current policy:
resp1 = await fetch(attestURL + 'operations/policy/current?api-version=2018-09-01-preview&tee=' + attestationType, { 'headers': { 'Authorization': token } });
await resp1.json();

// Update policy - Step 1:
resp2 = await fetch(attestURL + 'operations/policy/updatepolicy?api-version=2018-09-01-preview&tee=' + attestationType, {
  'headers': { 'accept': 'application/json, text/javascript, */*; q=0.01', 'authorization': token, 'content-type': 'application/json' },
  'body': encodedPolicy,
  'method': 'POST'
});
const resp2data = await resp2.json();
// Update policy - Step 2:
resp3 = await fetch(attestURL + 'operations/policy/current?api-version=2018-09-01-preview&tee=' + attestationType, {
  'headers': { 'accept': 'application/json, text/javascript, */*; q=0.01', 'authorization': token, 'content-type': 'application/json' },
  'body': resp2data,
  'method': 'PUT'
});
await resp3.json();

// Check Update Results:
resp4 = await fetch(attestURL + 'operations/policy/current?api-version=2018-09-01-preview&tee=' + attestationType, { 'headers': { 'Authorization': token } });
await resp4.json();

```

This is "Attest URI" that you'll update <br/>(use yours URI please):
---
> ![image](https://user-images.githubusercontent.com/13743374/279639971-7aee5dae-910a-4313-a98e-1dc95391ce29.png)


This is policy content that might be updated <br/>(use yours policy text please):
---
> ![image](https://user-images.githubusercontent.com/13743374/279636319-4e536bf0-3db8-441b-b131-3cd206fa2e80.png)

This is Token that must be updated to real value <br/>(use yours token please):
---
> ![image](https://user-images.githubusercontent.com/13743374/279637491-748b0a78-6eab-43bb-92c7-effca2431c86.png)
