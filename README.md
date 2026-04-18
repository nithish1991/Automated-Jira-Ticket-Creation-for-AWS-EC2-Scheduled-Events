# Automated-Jira-Ticket-Creation-for-AWS-EC2-Scheduled-Events
Automated Jira Ticket Creation for AWS EC2 Scheduled Events
This post describes the architecture and configuration required to automatically generate Jira Service Management (JSM) tickets in response to AWS Health notifications about scheduled EC2 instance events (e.g., reboots, instance retirements).

The aim is to eliminate the manual process of reviewing and forwarding AWS notification emails to a service desk email address. With this automation, you can proactively track and resolve AWS‑initiated maintenance activities and also we can do this maintenace under our control.

Architecture Overview
AWS Health Events → EventBridge → SQS → Jira Service Management Connector
<img width="800" height="507" alt="image" src="https://github.com/user-attachments/assets/f1a57fbc-ecf9-4324-9b29-955834b24ed3" />
At a high level:

AWS Health generates events for EC2 scheduled maintenance.
Amazon EventBridge captures and filters those events.
An Amazon SQS queue receives the events.
The Jira Service Management Connector consumes messages from SQS and creates issues in a JSM project.

AWS Health Setup
Using AWS Organizations, you can centralize scheduled AWS Health maintenance notifications for all EC2 instances across all member accounts into a single “central operations” account. From this account, events are visible in the AWS Health Dashboard and can be routed to other AWS services.

To delegate AWS Health Dashboard organization view to a central account, run the following command from your AWS Organizations management (root) account:
aws organizations register-delegated-administrator \ --account-id <CENTRAL_OPERATIONS_ACCOUNT_ID> \ --service-principal health.amazonaws.com

Replace <CENTRAL_OPERATIONS_ACCOUNT_ID> with the ID of the account where you want to centralize AWS Health events.

AWS EventBridge
EventBridge is used to capture EC2‑related AWS Health events and route them to an SQS queue.

Setup Steps
In your central operations account, create an EventBridge rule, for example: aws_schedule_maintenance_event.
Set the target of this EventBridge rule to an Amazon SQS queue that will buffer health events for downstream processing.

Delivering Events to Jira Service Management via SQS
The SQS queue receives the AWS Health event payloads from EventBridge. These messages are then delivered into a Jira Service Management project using the AWS Service Management Connector for Jira Service Management (often referred to as the “AWS JSM Connector”).

You can install the connector from the Atlassian Marketplace and configure it to read from the SQS queue and create issues in a chosen JSM project (for example, an infrastructure or operations service desk).

High‑Level Configuration in Jira
Once the AWS JSM Connector is installed:

Navigate to the AWS Service Management Connector configuration in Jira.
Configure an AWS account connection that has permissions to read from the SQS queue and interact with the necessary AWS services.
On the connector settings page, ensure the AWS Health integration is enabled.
Enable the desired Jira Service Management project and configure:

For details on accessing Forge‑based Jira app logs for troubleshooting, see Atlassian’s documentation: Access app logs

Outcome
With the above workflow implemented correctly (and with appropriate AWS and Jira configuration), AWS Health events for EC2 maintenance automatically generate Jira tickets in your chosen JSM project.

This gives operations teams:

Proactive visibility into upcoming instance retirements and maintenance.
A consistent ticketing trail for AWS‑initiated changes.
The ability to track remediation work through normal incident or request workflows.

Troubleshooting
Occasionally, the connection between the SQS queue and the JSM connector can fall out of sync (for example, due to transient network or authentication issues). When that happens, EC2 Health events may accumulate in the SQS queue without being turned into Jira tickets.

A typical temporary fix looks like this:

Check the SQS queue
Reset / re‑sync the connector in Jira

After the connection is re‑established and the sync completes, the SQS messages should start being processed, and Jira tickets should appear for any pending EC2 instance retirement or maintenance events.

Future Improvements
To make the automation more effective and user‑friendly, you can enrich the Jira tickets with additional context:

Include instance names in the ticket summary or description, not just the instance ID.
Add environment tags (e.g., prod, staging) and application identifiers to help teams quickly understand impact.
Use a Lambda function or similar service to:

Enriching tickets in this way makes automated notifications more actionable and reduces the time needed for responders to identify and remediate affected resources.

Further this project is complete open source, we can create jira tickets for AWS scheduled maintenance for any AWS services using AWS health.
