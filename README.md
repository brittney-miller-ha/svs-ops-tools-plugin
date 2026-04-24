# SVS Tools Plugin

Claude skills for the **Specialized Vendor Services (SVS)** team at Instacart.

## Skills

### /file-svs-ticket
Files a new SVS Jira ticket following SVS best practices and the team's filing rubric.

**What it does:**
- Gathers all required fields: work type, components, priority, and due date
- Suggests a priority (P0–P4) based on context from your message, Slack, emails, or meeting notes
- Recommends a parent ticket by searching open SVS Initiatives and Epics
- Drafts a description using the SVS template: Context → Actions to be Completed → Conditions of Satisfaction
- Automatically tags every ticket with the `Claude-generated Ticket` component
- Reminds you to open and verify the ticket after filing

**Trigger phrases:**
- "File a Jira ticket"
- "Create a ticket for..."
- "Can you track that?"
- "Let's make a ticket for this"

## References
- [SVS Jira Best Practices](https://instacart.atlassian.net/wiki/spaces/cx/pages/5486739635/SVS+-+Jira+Best+Practices)
- [[WIP] SVS Jira Filing Guidelines](https://docs.google.com/document/d/1rmXc0pGEkHAD3RsQmOgnWXHdqnJULA74EtXDCha5CX8/edit)

## Maintained by
Brittney Miller — brittney.miller@instacart.com
