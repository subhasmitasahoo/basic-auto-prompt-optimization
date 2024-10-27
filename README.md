# Auto Prompt Optimization

## Algorithm with simple steps
```
Here's how Auto-Prompt Optimization works in simple steps:
1. Start Simple
   * Begin with a basic 2-line prompt
   * Just include the task and class names (no detailed descriptions needed!)
2. Iterate and Improve
   * Test the initial prompt and measure accuracy
   * Create a "gradient prompt" asking the model for improvement feedback
   * Generate new prompt versions based on this feedback
   * Test these new prompts and keep the best performers
3. Repeat and Perfect
   * Run multiple optimization rounds
   * Select the highest-performing prompt
```
Refer to https://arxiv.org/pdf/2305.03495 for more details.

## Code
I think you should give it a try using the shared algorithm, prompt templates and data, so not sharing any code! 
Please feel free to add a comment if you have any questions on implementation.

## Prompts

### Sample initial prompt
```
#Task
Categorize the custom support ticket.

# Output format
Classify into one of these classes: 'Technical Support', 'Billing', 'General information', 'Complaint and escalations', 'Feedback and suggestions'.
```

### Gradient prompt template
```
I'm trying to write a zero-shot classifier prompt.
My current prompt is:
"{prompt}"

But this prompt gets the following examples wrong:
{error_string}

Give {num_feedbacks} reasons why the prompt could have gotten these examples wrong.
Wrap each reason with <START> and <END>
```

### Edit prompt template
```
I'm trying to write a zero-shot classifier.
My current prompt is:
"{prompt}"

But it gets the following examples wrong:
{error_str}

Based on these examples the problem with this prompt is that {gradient}

Based on the above information, write {steps_per_gradient} different improved prompts.
To handle the losses, consider making the descriptions of each class in the improved prompts.
Each prompt should be wrapped with <START> and <END>.
The {steps_per_gradient} new prompts are:
```

## Sample Training/Eval data for a dry run
```
TRAINING_DATA = [
        # Technical Support
    Example("I can't log into my account, it keeps saying 'invalid credentials' even though I'm sure my password is correct.", "Technical Support"),
    Example("The app keeps crashing whenever I try to upload a photo. I'm using the latest iPhone version.", "Technical Support"),
    Example("My dashboard isn't showing any of my recent activity from the past week. Is there a system issue?", "Technical Support"),
    Example("The export function isn't working - when I click 'download report' nothing happens.", "Technical Support"),
    Example("Getting a 404 error when trying to access my profile settings. Please help!", "Technical Support"),
    Example("The sync feature between my mobile and desktop isn't working properly.", "Technical Support"),
    Example("I'm unable to reset my two-factor authentication because I lost my phone.", "Technical Support"),
    Example("The website is extremely slow and takes forever to load my inventory page.", "Technical Support"),
    Example("I keep getting logged out every few minutes, even when I check 'remember me'.", "Technical Support"),
    Example("Can't seem to connect my Google Calendar to the platform. Getting an integration error.", "Technical Support"),
    Example("My dual-monitor setup suddenly stopped working after the latest software update v2.3.5. The primary display works fine at 4K resolution, but the secondary monitor shows artifacts and flickers when I try to drag windows across. I've already tried updating my graphics drivers and rolling back the update.", "Technical Support"),
    Example("We're experiencing intermittent connectivity issues with the API integration between your platform and our custom CRM system. The error logs show timeout exceptions occurring specifically during high-traffic periods (2000+ simultaneous requests), but only when executing complex queries with multiple JOIN operations.", "Technical Support"),
    Example("After migrating our database from PostgreSQL 12 to 13, the real-time analytics dashboard is showing inconsistent data. The discrepancy seems to affect only aggregated metrics from the last 30 days, while historical data and raw data views appear correct.", "Technical Support"),
    Example("The automated backup system is creating corrupted incremental backups when the file size exceeds 2GB. We've noticed this happens specifically with files containing special characters in their names and when the backup process coincides with our nightly maintenance window.", "Technical Support"),
    Example("Since implementing the SSO integration with Okta, users from our European offices are experiencing 20-30 second delays during authentication, but only when accessing through our VPN. The same setup works instantly for our US-based employees.", "Technical Support"),

    # Billing
    Example("I was charged twice for my monthly subscription. Please refund the extra payment.", "Billing"),
    Example("When will my refund for order #45789 be processed? It's been 5 days.", "Billing"),
    Example("I need to update my credit card information for automatic payments.", "Billing"),
    Example("Can you explain the charges on my latest invoice? There's an item I don't recognize.", "Billing"),
    Example("I cancelled my subscription but was still charged this month.", "Billing"),
    Example("Need a copy of all my invoices from the last financial year for tax purposes.", "Billing"),
    Example("The system won't accept my new debit card, keeps saying invalid card number.", "Billing"),
    Example("I was promised a discount but it wasn't applied to my last bill.", "Billing"),
    Example("How do I change my billing cycle from monthly to annual?", "Billing"),
    Example("Need to update the billing address on my account for tax purposes.", "Billing"),
    Example("Our organization recently underwent a merger, and we need to consolidate billing for 3 separate enterprise accounts (IDs: #ERF456, #ERF789, #ERF012) while maintaining separate usage analytics and access controls. We also need to ensure compliance with both EU and US tax regulations.", "Billing"),
    Example("There appears to be a discrepancy in the usage-based billing calculation for our API calls. According to our internal monitoring, we made 2.3M calls last month, but we're being charged for 2.8M. We need a detailed breakdown of the calculation methodology and timestamp-level access logs for reconciliation.", "Billing"),
    Example("We've been grandfathered into a custom enterprise plan from 2019 ($50k/year) but noticed that the latest invoice reflects the current market rate ($75k/year). Our contract (ref: ENTX-2019-443) specifically includes a 5-year price lock guarantee with a maximum 3% annual increase.", "Billing"),
    Example("Need to implement a complex departmental billing structure where R&D expenses are split 70/30 between two cost centers, marketing is billed to three different regions based on usage patterns, and admin costs need to be amortized across all departments using a custom formula.", "Billing"),
    Example("We're transitioning from a corporate credit card to ACH payments mid-billing cycle while simultaneously switching from monthly to quarterly billing. Need to ensure this won't affect our pre-negotiated volume discounts or interrupt service to our 1500+ users.", "Billing"),
        
    # General Information
    Example("What are your business hours during the holiday season?", "General Information"),
    Example("Can you tell me more about your enterprise plan features?", "General Information"),
    Example("Do you offer student discounts on your premium packages?", "General Information"),
    Example("What's the typical processing time for standard shipping?", "General Information"),
    Example("Are your products available for international shipping?", "General Information"),
    Example("Could you explain how the reward points system works?", "General Information"),
    Example("What file formats do you support for uploads?", "General Information"),
    Example("Do you have any upcoming maintenance schedules?", "General Information"),
    Example("What's the maximum file size limit for attachments?", "General Information"),
    Example("Can you explain the difference between your basic and premium plans?", "General Information"),
    Example("Could you provide detailed documentation about your platform's compliance with GDPR, CCPA, HIPAA, and SOC 2 Type II requirements? We specifically need information about data residency options for our APAC clients and your roadmap for upcoming ISO 27701 certification.", "General Information"),
    Example("What are the exact specifications and limitations of your API rate limiting system? We're particularly interested in understanding how concurrent request handling differs between your various enterprise tiers, and whether these limits are adjustable for specific endpoints or time windows.", "General Information"),
    Example("Can you explain the architectural differences between your multi-region deployment options? We need to understand the implications for data replication latency, failover mechanisms, and how this affects our SLA, particularly for our real-time processing requirements.", "General Information"),
    Example("What's your disaster recovery protocol for scenarios involving cascading failures across multiple availability zones? We need details about your RPO and RTO guarantees, especially regarding the preservation of transactional data integrity during forced failovers.", "General Information"),
    Example("Could you break down the environmental impact metrics of using your cloud services versus on-premise solutions? We need this information for our annual sustainability report, specifically focusing on power usage effectiveness (PUE) and carbon offset programs.", "General Information"),
    
    # Complaint and Escalations
    Example("I've been trying to resolve this issue for weeks and no one is helping. I want to speak to a supervisor.", "Complaint and Escalations"),
    Example("This is the third time I'm reporting this bug and it's affecting my business operations. Urgent resolution needed!", "Complaint and Escalations"),
    Example("Your support team has been extremely unhelpful and I'm considering canceling my subscription.", "Complaint and Escalations"),
    Example("I demand immediate attention to this issue as it's causing significant financial loss to my company.", "Complaint and Escalations"),
    Example("This is unacceptable! I've been waiting for 2 hours on hold and no one has picked up.", "Complaint and Escalations"),
    Example("I want to file a formal complaint about your service quality in the past month.", "Complaint and Escalations"),
    Example("Your product is not delivering what was promised during sales. I need this escalated immediately.", "Complaint and Escalations"),
    Example("Multiple failed attempts to resolve this - requesting urgent escalation to management.", "Complaint and Escalations"),
    Example("The level of service I've received is completely below standard. Need management intervention.", "Complaint and Escalations"),
    Example("This is my final attempt to resolve this before involving consumer protection services.", "Complaint and Escalations"),
    Example("This is absolutely unacceptable! We've experienced 4 critical outages in the past month, each lasting over 3 hours, causing us to miss our SLA commitments to our tier-1 clients. We've documented $375,000 in lost revenue, and your standard compensation offer of 10% credit is insulting. I need to speak with your Chief Operating Officer immediately.", "Complaint and Escalations"),
    Example("I've spent 47 hours over the past two weeks working with 12 different support representatives, none of whom seem to understand the severity of our authentication system failure. This has forced us to manually provision access for 3,000+ employees, violating our security protocols. If this isn't resolved by end of day, we'll be forced to initiate legal proceedings.", "Complaint and Escalations"),
    Example("Your latest 'enhanced' security update has completely broken our mission-critical automated workflow systems that we spent 8 months building based on your API documentation. Despite 5 escalations and 2 conference calls with your technical team, we're still without a resolution or even a proper explanation. This is severely damaging our reputation with our clients.", "Complaint and Escalations"),
    Example("We were promised enterprise-grade support with a 15-minute response time for critical issues, yet we've been waiting for 72 hours for a response to a severity-1 ticket about database corruption. Our entire EMEA operation is at a standstill, affecting 140,000 end-users. This is a breach of contract and we're documenting all losses.", "Complaint and Escalations"),
    Example("After migrating our entire infrastructure to your platform based on promises made during the sales process, we've discovered that half of the promised features either don't exist or are 'coming soon'. We've invested $2.3M in this migration and need immediate resolution or we'll be forced to pursue all available legal remedies while publicly documenting our experience.", "Complaint and Escalations"),
        
    # Feedback and Suggestions
    Example("It would be great if you could add a dark mode option to the dashboard.", "Feedback and Suggestions"),
    Example("Consider adding bulk upload functionality - it would save us a lot of time.", "Feedback and Suggestions"),
    Example("The new UI is much better, but the search function could be more prominent.", "Feedback and Suggestions"),
    Example("Your customer service team was excellent today, especially Sarah who helped me.", "Feedback and Suggestions"),
    Example("A mobile app would make your service much more accessible and convenient.", "Feedback and Suggestions"),
    Example("The latest update has made the platform much faster - great improvement!", "Feedback and Suggestions"),
    Example("Would love to see integration with more third-party tools in the future.", "Feedback and Suggestions"),
    Example("The new reporting feature is fantastic, but could use more export options.", "Feedback and Suggestions"),
    Example("Your onboarding process was smooth, but a video tutorial would be helpful.", "Feedback and Suggestions"),
    Example("Really appreciate the recent changes to the notification system - much clearer now.", "Feedback and Suggestions"),
    Example("The recent implementation of WebAuthn for passwordless authentication is a step in the right direction, but it could be significantly improved by adding conditional step-up authentication for high-risk operations and integration with hardware security keys. Also, consider adding biometric authentication options for mobile users with detection of device integrity status.", "Feedback and Suggestions"),
    Example("While your GraphQL API is powerful, the developer experience could be enhanced by implementing real-time subscription support for certain endpoints, adding field-level performance metrics, and providing better tooling for query optimization. The current playground lacks important features like query persistence and team sharing capabilities.", "Feedback and Suggestions"),
    Example("Your kubernetes operator is quite robust, but would benefit from adding support for custom resource definitions that allow for more granular control over pod scaling behaviors. Additionally, implementing automatic certificate rotation and secret management integration would significantly improve the security posture.", "Feedback and Suggestions"),
    Example("The analytics dashboard is comprehensive, but could be improved by adding support for custom retention cohort analysis with multiple attribution models. It would also be beneficial to have the ability to create custom metrics using a SQL-like interface and schedule automated exports with dynamic parameters.", "Feedback and Suggestions"),
    Example("Your machine learning pipeline integration is promising, but needs better support for A/B testing frameworks, automated model retraining triggers based on drift detection, and more sophisticated feature store capabilities. Consider adding integration with popular ML ops tools and support for distributed training across multiple zones.", "Feedback and Suggestions")
]
```
