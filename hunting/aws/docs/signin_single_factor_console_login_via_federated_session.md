# Signin Single Factor Console Login via Federated Session

---

## Metadata

- **Author:** Elastic
- **Description:** This hunting query identifies when a user signs in to the AWS Management Console using a federated session and single-factor authentication. Federated sessions are typically used by users who are not directly managed by AWS IAM or have temporary credentials. Single-factor authentication without MFA may indicate a security risk, as MFA adds an additional layer of security to the authentication process. This could also indicate the abuse of STS tokens to bypass MFA requirements.

- **UUID:** `953b1252-5efd-11ef-a997-f661ea17fbce`
- **Integration:** [aws.cloudtrail](https://docs.elastic.co/integrations/aws/cloudtrail)
- **Language:** `[ES|QL]`
- **Source File:** [Signin Single Factor Console Login via Federated Session](../queries/signin_single_factor_console_login_via_federated_session.toml)

## Query

```sql
from logs-aws.cloudtrail-*
| where @timestamp > now() - 7 day
| where
    event.provider == "signin.amazonaws.com"
    and event.action == "GetSigninToken"
    and aws.cloudtrail.event_type == "AwsConsoleSignIn"
    and aws.cloudtrail.user_identity.type == "FederatedUser"
| dissect aws.cloudtrail.additional_eventdata "{%{?mobile_version_key}=%{mobile_version}, %{?mfa_used_key}=%{mfa_used}}"
| where mfa_used == "No"
```

## Notes

- Use the `aws.cloudtrail.user_identity.type` field to identify the user type making the request. Federated users are typically given temporary credentials to access AWS services.
- `aws.cloudtrail.user_identity.session_context.session_issuer.arn` field represents the ARN of the IAM entity that created the federated session. This IAM entity could be compromised and used to create federated sessions.

## MITRE ATT&CK Techniques

- [T1078.004](https://attack.mitre.org/techniques/T1078/004)

## License

- `Elastic License v2`