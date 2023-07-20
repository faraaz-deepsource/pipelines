# Triggering pipelines remotely

Pipelines can be triggered remotely using the following methods for the following providers:

## Github

ref: https://github.com/peter-evans/repository-dispatch/blob/main/src/main.ts 

```js
import * as core from '@actions/core'
import * as github from '@actions/github'
import {inspect} from 'util'

/* eslint-disable  @typescript-eslint/no-explicit-any */
function hasErrorStatus(error: any): error is {status: number} {
  return typeof error.code === 'number'
}

function getErrorMessage(error: unknown) {
  if (error instanceof Error) return error.message
  return String(error)
}

async function run(): Promise<void> {
  try {
    const inputs = {
      token: core.getInput('token'),
      repository: core.getInput('repository'),
      eventType: core.getInput('event-type'),
      clientPayload: core.getInput('client-payload')
    }
    core.debug(`Inputs: ${inspect(inputs)}`)

    const [owner, repo] = inputs.repository.split('/')

    const octokit = github.getOctokit(inputs.token)

    await octokit.rest.repos.createDispatchEvent({
      owner: owner,
      repo: repo,
      event_type: inputs.eventType,
      client_payload: JSON.parse(inputs.clientPayload)
    })
  } catch (error) {
    core.debug(inspect(error))
    if (hasErrorStatus(error) && error.status == 404) {
      core.setFailed(
        'Repository not found, OR token has insufficient permissions.'
      )
    } else {
      core.setFailed(getErrorMessage(error))
    }
  }
}

run()
```

## Gitlab

ref: https://docs.gitlab.com/ee/ci/triggers/#use-a-webhook

```bash
curl --request POST \
  --form token=TOKEN \
  --form ref=main \
  --form "variables[UPLOAD_TO_S3]=true" \
  "https://gitlab.example.com/api/v4/projects/123456/trigger/pipeline"
```

## BitBucket

ref: https://developer.atlassian.com/cloud/bitbucket/rest/api-group-pipelines/#api-repositories-workspace-repo-slug-pipelines-post

```python
# This code sample uses the 'requests' library:
# http://docs.python-requests.org
import requests
import json

url = "https://api.bitbucket.org/2.0/repositories/{workspace}/{repo_slug}/pipelines"

headers = {
  "Accept": "application/json",
  "Content-Type": "application/json"
}

payload = json.dumps( {
  "type": "<string>",
  "uuid": "<string>",
  "build_number": 33,
  "creator": {
    "type": "<string>"
  },
  "repository": {
    "type": "<string>"
  },
  "target": {
    "type": "<string>"
  },
  "trigger": {
    "type": "<string>"
  },
  "state": {
    "type": "<string>"
  },
  "created_on": "<string>",
  "completed_on": "<string>",
  "build_seconds_used": 50
} )

response = requests.request(
   "POST",
   url,
   data=payload,
   headers=headers
)

print(json.dumps(json.loads(response.text), sort_keys=True, indent=4, separators=(",", ": ")))
```
