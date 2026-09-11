### Access control vulnerabilities in multi-step processes

Many websites implement important functions over a series of steps. This is common when:

- A variety of inputs or options need to be captured.
- The user needs to review and confirm details before the action is performed.

For example, the administrative function to update user details might involve the following steps:

1. Load the form that contains details for a specific user.
2. Submit the changes.
3. Review the changes and confirm.

Sometimes, a website will implement rigorous access controls over some of these steps, but ignore others. Imagine a website where access controls are correctly applied to the first and second steps, but not to the third step. The website assumes that a user will only reach step 3 if they have already completed the first steps, which are properly controlled. An attacker can gain unauthorized access to the function by skipping the first two steps and directly submitting the request for the third step with the required parameters.

Steps to reproduce:

