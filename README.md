# OCI ARM instance provisioning

This fork uses [provision-arm.yml](.github/workflows/provision-arm.yml) to request one Oracle Cloud `VM.Standard.A1.Flex` instance.

| Setting | Value |
| --- | --- |
| OCPUs | 2 |
| Memory | 12 GB |
| Boot volume | 200 GB |
| Instance name | `oci-free-arm` |
| Retry interval inside a running job | 600 seconds between attempt starts |
| Maximum attempts per run | 35 |
| Job timeout | 350 minutes |

The boot volume uses the full 200 GB combined Always Free boot/block storage allowance if no other volumes consume it. Use the home region and check existing usage and eligibility in Oracle Cloud before changing resources.

## Start and inspect retries

1. Open [Actions → OCI ARM Retry](https://github.com/glistenlau/oci-free-arm-instance/actions/workflows/provision-arm.yml).
2. Select **Run workflow**, choose `main`, and submit.
3. Open the run, select `create-vm`, and expand **Retry every 10 minutes, stopping when a VM exists**.
4. Look for timestamped `Attempt 1/35`, `Attempt 2/35`, and `Next attempt ... in ...s` messages.

Retries happen inside the same run. The run remains **In progress** while sleeping between attempts; it does not create a new run for every attempt. You can close the browser after starting it.

The cron expression `4,14,24,34,44,54 * * * *` also requests new runs. GitHub schedules are best effort and may be delayed or dropped. A saved cron expression or a successful manual run does not prove that scheduled triggers are working. Check for runs labeled **Scheduled**. If a retry window ends and no scheduled run follows, use **Run workflow** to start another window.

[GitHub scheduling troubleshooting](https://docs.github.com/en/actions/how-tos/troubleshoot-workflows#scheduled-workflows-running-at-unexpected-times) · [GitHub runner limits](https://docs.github.com/en/actions/reference/limits)

## Results and stopping

- **Out of host capacity:** wait until the next attempt in the same run.
- **VM creation accepted:** retries stop. Confirm the instance state and public IP in Oracle Cloud.
- **An A1 instance already exists:** no additional instance is requested. The inventory check runs before every attempt and stops if it fails.
- **Other errors:** the job fails immediately with sanitized error details. Fix the configuration before restarting.
- **Retry window finished:** all 35 attempts completed without capacity. A new run is needed.

A green Actions status alone does not mean a VM was created; read the summary. Concurrency prevents two provisioning jobs from running at once.

To stop now, cancel the active run. To prevent future runs, disable the workflow from its menu or set the repository variable `OCI_AUTOMATION_ENABLED` to `false`. Changing the variable does not cancel an already-running job.

## Configuration

Store these ten repository secrets under **Settings → Secrets and variables → Actions**:

| Secret | Contents |
| --- | --- |
| `OCI_CLI_USER` | OCI user OCID |
| `OCI_CLI_TENANCY` | Tenancy OCID |
| `OCI_CLI_FINGERPRINT` | Fingerprint matching the API private key |
| `OCI_CLI_REGION` | Home-region identifier |
| `OCI_CLI_KEY_CONTENT` | Complete API private key in PEM format |
| `OCI_COMPARTMENT_ID` | Target compartment OCID |
| `OCI_SUBNET_ID` | Public subnet OCID |
| `AD_NAME` | Availability domain name |
| `IMAGE_ID` | Compatible ARM image OCID in the same region |
| `SSH_PUBLIC_KEY` | SSH public key; keep its private key on your own machine |

Set the repository **variable** `OCI_AUTOMATION_ENABLED` to `true` only after the credentials and networking are ready. The subnet needs an internet-gateway route; restrict SSH ingress to your own IP/CIDR.

Credentials are read from Actions secrets and stored in temporary files with restrictive permissions, then removed when the step exits. This fork reports results in Actions logs and summaries; it does not send Discord notifications.

## Credits and license

Adapted from [maoucodes/oci-free-arm-instance](https://github.com/maoucodes/oci-free-arm-instance), with upstream work by heykapil. See [LICENSE](LICENSE).
