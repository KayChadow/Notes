# Access Control
Access control determines whether the user is allowed to carry out the action that they are attempting to perform. 

## Privilege Escalation
Privilege escalation is all about exploiting the access control mechanism in a way that allows you to gain access to admin functionality, or other users' acounts/actions.

### Vertical
Reaching admin functionalities, or at least higher privileges than you get as a normal user.

Misconfigurations are crucial. Maybe the only admin protection is by obscuring the urls (BAD). Maybe permissions are decided by user controllable parameters (BAD). Maybe you can use headers like `X-Original-URL` or `X-Rewrite-URL` to bypass controls that forbid you from accessing certain endpoints.

### Horizontal
As a user you can access your own account, and do actions in name of your own account. Horizontal privilege escalation refers to a type of vulnerability that gives an attack access to other similar priviled users. Sometimes other users have admin rights, and thus it results in vertical privilege escalation.

## IDOR
Insecure direct object references (IDOR) are a type of access control vulnerability that arises when an application uses user-supplied input to access objects directly. IDOR vulnerabilities are most commonly associated with horizontal privilege escalation, but they can also arise in relation to vertical privilege escalation.
It arises when you have some pattern like `?customer_number=278341`, you can maybe just change it.

## More Vulnerabilities

### Multi Step
Sometimes an action that can be performed by users contain multiple steps. Like selecting, changing, confirming or something. But when not all parts have proper access control, and attacker can target these vulnerable endpoints by directly submitting tha requests.

### Referer Based
Sometimes the access control of admin subpages depend on the `Referer` header containing a page that has working access controls. But this is bad, since you can just change the referer header to the required page.

### Location based
Some websites enforce access controls based on the user's geographical location. This can apply, for example, to banking applications or media services where state legislation or business restrictions apply. These access controls can often be circumvented by the use of web proxies, VPNs, or manipulation of client-side geolocation mechanisms.

## How to prevent access control vulnerabilities
Access control vulnerabilities can be prevented by taking a defense-in-depth approach and applying the following principles:

- Never rely on obfuscation alone for access control.
- Unless a resource is intended to be publicly accessible, deny access by default.
- Wherever possible, use a single application-wide mechanism for enforcing access controls.
- At the code level, make it mandatory for developers to declare the access that is allowed for each resource, and deny access by default.
- Thoroughly audit and test access controls to ensure they work as designed.

# Sources
- Portswigger Academy: https://portswigger.net/web-security/access-control 