---
title: Building a Unified View of the Customer in Data Cloud
layout: blog
category: [Salesforce, Data Cloud, Customer360]
excerpt: In this blog, I share my practical lessons and approach on how to handle a Data Cloud Implementation around creating a Unified View of a Customer.
comments: true
---

Organizations today face a pressing challenge: customer data exists everywhere, yet nowhere all at once. Marketing knows engagement patterns. Sales understands purchase intent. Service tracks support history. E-commerce has transaction data. But bringing these fragments together into a single, actionable view - one that actually drives business outcomes - requires far more than flipping a switch.

Salesforce Data Cloud offers powerful capabilities to create that unified customer view, but the path to sucsscess isn't about speed. It's about precision, planning, and patience. Done right, a unified customer view becomes the foundation for personalization, intelligent automation, and genuine customer understanding. Done hastily, it becomes another data project that fails to deliver on its promise.

## Start with the End in Mind: Define What Your Unified Model Should Achieve

Before ingesting a single data source, step back and ask the fundamental question: **what business outcomes should this unified view enable?**

**The answer shapes everything that follows.** A marketing team seeking to deliver hyper-personalized campaigns needs different data points than a service organization trying to predict customer churn. A retail business building omnichannel experiences requires different real-time data than a financial services firm focused on risk assessment.

**Document clear use cases upfront.** 
- Which departments will consume this unified view?
- What decisions will they make with it? 
- How will success be measured?

These aren't abstract questions - they determine which systems to integrate, which data points matter, and how fresh that data needs to be. Starting with two or three focused use cases allows you to demonstrate value early while building the foundation for expansion.

This strategic alignment phase typically takes four to six weeks, but it prevents months of rework later. Cross-functional stakeholders - from IT and legal to sales and marketing - need to weigh in on requirements, governance policies, and success metrics. Their input ensures the unified view serves actual business needs rather than becoming a technically impressive but practically unusable data warehouse.

## Map the Data Landscape: Understanding What Systems Hold Which Truth

Once business objectives are clear, conduct a thorough audit of your data landscape. This means identifying every system that holds customer information and understanding what each one contributes to the complete picture.

Your CRM might be the system of record for basic customer details. Marketing Cloud tracks engagement and preferences. E-commerce platforms hold purchase behavior. Service systems contain interaction history. External data sources provide enrichment. Each system has its own schema, naming conventions, and data quality standards.

**Create detailed data dictionaries for each source.** 
- What fields exist? 
- How are they formatted? 
- What do they actually mean in business context? 

This documentation becomes essential when you begin the harmonization process. It also reveals gaps, overlaps, and conflicts - discovering that "customer ID" in one system means something entirely different than "client code" in another.

This assessment phase uncovers which data is clean and ready versus which requires significant cleanup. It identifies which systems update in real-time versus batch processes. It highlights where master data lives versus where it's duplicated or derived. Understanding this landscape before you start building prevents costly architectural decisions made on false assumptions.

## Design the Integration Strategy: DLOs, Custom DLOs, and Harmonization Through DMOs

Data Cloud's architecture follows a clear progression: raw data flows via Data Streams, gets transformed and stored in Data Lake Objects (DLOs), and then maps to standardized Data Model Objects (DMOs) that create the unified view.

**Leverage Standard DLOs First**

Data Cloud provides pre-built DLOs and DMOs designed to accommodate the vast majority of use cases. These 89+ standard DMOs cover party information, engagement data, transactions, loyalty programs, and more. Using these standard models accelerates implementation and ensures compatibility with Data Cloud's identity resolution and segmentation capabilities.

Map your source data to these canonical models wherever possible. Yes, your organization has unique nuances, but forcing every detail into custom objects creates maintenance overhead and limits your ability to leverage Data Cloud's built-in intelligence.

**Build Custom DLOs for Unique Business Context**

That said, every organization has data that doesn't fit neatly into standard models. Industry-specific attributes, proprietary scoring systems, custom product hierarchies - these require custom DLOs. The key is being deliberate about when customization serves a genuine business need versus when it's just preference for familiar naming conventions.

Custom DLOs enable you to preserve business logic during transformation. You might need to join data from multiple sources, apply calculations, or create derived attributes before mapping to DMOs. Batch data transforms allow this type of data preparation, ensuring that what flows into your DMOs is already cleaned, enriched, and business-ready.

**Harmonization Through DMOs Creates the Single Source of Truth**

The real magic happens when DLOs map to DMOs. This harmonization process takes disparate data from different sources and unifies it into a consistent format. An email address from your marketing system, a customer ID from your CRM, and a loyalty number from your e-commerce platform all get mapped to standardized fields in the Individual DMO.

Data Cloud uses Fully Qualified Keys to avoid conflicts when data from multiple sources is harmonized. This technical foundation ensures that even when the same customer exists in five different systems with five different identifiers, the unified view maintains data lineage and relationships.

## Make Strategic Decisions About Data Freshness: Real-Time, Near Real-Time, and Batch

> Not all data needs to be real-time, and treating everything as urgent creates unnecessary cost and complexity.

**Real-Time Data**: Reserve this for information that drives immediate action. Website behavior that triggers personalized experiences. Transaction data that feeds fraud detection. Customer interactions that update service agent screens. Real-time processing ensures millisecond to second latency but requires more sophisticated infrastructure and incurs higher costs.

**Near Real-Time Data**: Many use cases work perfectly well with data that's minutes old rather than seconds. Campaign responses. Product availability updates. Support ticket status changes. Near real-time processing offers a middle ground—fresh enough for most personalization needs without the overhead of streaming architectures.

**Batch Processing**: Historical data, daily aggregations, and data warehouse updates work fine with scheduled batch processing. Monthly purchase summaries. Quarterly metrics. Data quality audits. Batch processing is simpler to implement, more resource-efficient, and handles large data volumes effectively.

Your data strategy should explicitly define which data points from which systems require which level of freshness. This decision directly impacts architecture, cost, and complexity. It also shapes how your unified view gets used - real-time activation requires real-time data, while analytical use cases tolerate batch updates.

## Execute Identity Resolution with Care: Creating True Unified Profiles

> Even perfectly harmonized data doesn't create a unified view until identity resolution connects the dots.

A single customer might appear in your systems as sarah.johnson@work.com in Marketing Cloud, Customer ID 847392 in your CRM, and loyalty number LTY-29384 in your e-commerce platform. Identity resolution finds these disparate records and links them into a single unified profile.

**Matching Rules Define the Connections**

Matching rules specify the criteria for identifying that different records represent the same person. You might match on exact email address. Or fuzzy match on name plus ZIP code. Or deterministic match on a customer number that appears across systems.

The sophistication of your matching rules depends on your data quality and business requirements. Deterministic matching provides precision - when records match on a specific identifier, you know with certainty they're the same person. Probabilistic matching casts a wider net, using patterns and statistical models to find likely matches even when identifiers don't align perfectly.

**Reconciliation Rules Resolve Conflicts**

When the same customer has different information in different systems - one source says they live in California, another says Texas - reconciliation rules determine which value to trust. You might prioritize the most recently updated record. Or trust specific systems over others. Or keep all versions and flag the conflict for review.

These rules shape the quality of your unified profile. Poor reconciliation logic creates profiles that contain outdated or incorrect information, undermining trust in the entire system.

**The Unified Profile Becomes Your Single Source of Truth**

When identity resolution runs successfully, it creates unified profiles that consolidate all information about each customer. These profiles maintain data lineage - you can see which source contributed each piece of information - while presenting a single, coherent view of the customer across all touchpoints.

## Design for Activation: Where Will This Unified View Actually Be Used?

> A unified customer view has no value sitting in Data Cloud unless it flows to the places where decisions get made and actions get taken.

**Activation Targets Define Distribution Channels**

Activation targets specify where you'll send segment data and unified profile information. Marketing Cloud for campaign orchestration. Advertising platforms for paid media. Sales Cloud for account insights. Service Cloud for agent context. External systems via APIs.

Configure these activation targets before you start creating segments and activations. Each target has its own requirements for data format, refresh frequency, and available attributes. Understanding these constraints upfront prevents discovering late in the process that critical data can't flow where it needs to go.

**Segmentation Enables Targeted Action**

Segmentation groups customers based on unified attributes, behaviors, and calculated insights. Top purchasers. At-risk customers. High-engagement prospects. Campaign responders. These segments become the basis for personalized experiences across channels.

The unified view makes segmentation dramatically more powerful. Instead of segmenting only on marketing data or only on purchase history, you can combine signals from across the customer journey. A segment might include customers who engaged with specific content, made purchases above a threshold, and haven't contacted support in 90 days - criteria that span three different source systems unified into a single profile.

**Activation Brings the Unified View to Life**

Activation takes those segments and pushes them to your configured targets along with the attributes needed for personalization. Email campaigns in Marketing Cloud receive not just a list of contacts but their preferences, purchase history, and engagement patterns. Sales teams see enriched account information drawn from multiple systems. Service agents get complete interaction history regardless of which channel the customer used.

The unified view transforms from a data asset into actual business value when it powers these activations.

## Establish Data Quality Standards: Garbage In Still Equals Garbage Out

> Even the most sophisticated data architecture fails if the underlying data is poor quality.

**Define Quality Metrics Upfront**

Establish clear, measurable standards for data quality before you begin ingestion. 
- What level of completeness is acceptable? 
- How accurate must key fields be? 
- How current should data be? 
- What degree of consistency is required across sources?

These metrics become the baseline for monitoring and improvement. They also help prioritize data cleansing efforts - not all data needs to be perfect, but critical fields that drive identity resolution and personalization require higher standards.

**Implement Automated Quality Checks**

Manual data quality reviews don't scale. Implement automated validation rules that flag errors at the point of entry. Configure data profiling tools to identify anomalies, duplicates, and inconsistencies. Set up alerts for data quality issues that breach defined thresholds.

Data Cloud includes quality monitoring capabilities, but these need to be configured based on your specific standards and business rules. 
- What constitutes a valid email format? 
- Which fields are mandatory versus optional? 
- How should date formats be standardized? 

Define these rules explicitly and enforce them automatically.

**Create Continuous Improvement Processes**

Data quality isn't a one-time project. Establish regular audits that assess quality metrics. Create feedback loops that allow users to report data issues. Assign data stewards responsible for maintaining quality standards in their domains.

As the unified view gets used, quality issues become apparent - duplicates that identity resolution missed, outdated information that wasn't refreshed, mismatches between related records. These discoveries should feed back into improved data cleansing, better matching rules, and refined reconciliation logic.

## Accept the Reality: Doing It Right Takes Time

A Basic Data Cloud implementation takes 12 to 24 weeks for most organizations. More complex environments - multiple clouds, dozens of data sources, sophisticated customization requirements - often need six to 18 months.

That timeline reflects real work that can't be shortcut without consequences:

- **Discovery and assessment**: understanding your data landscape and defining requirements 
- **Solution design**: architecting the technical approach and data model
- **Development and build**: configuring integrations, building custom DLOs, implementing identity resolution
- **Testing and validation**: ensuring data quality, accuracy, and performance
- **Deployment and refinement**: going live and iterating based on real-world usage

Organizations that try to compress these timelines inevitably face issues. Poor matching rules that create duplicate profiles instead of unified ones. Data quality problems that undermine trust in the system. Architectural decisions that don't scale. Missing use cases that require rework.

**The pressure to show quick results is real. Stakeholders want to see value from their Data Cloud investment. But the fastest path to value isn't the shortest path to completion - it's starting with focused use cases, building a solid foundation, and expanding thoughtfully.**

## The Payoff: A Unified View That Actually Works

When you take the time to do this right, the unified customer view becomes transformative.

Marketing delivers campaigns with 3x better engagement because they're based on complete customer understanding, not fragmented data. Sales teams close deals faster because they see the full context of customer relationships. Service agents resolve issues on first contact because they have immediate access to interaction history across all channels.

The unified view enables AI and automation that actually works - because it's trained on accurate, complete, consistent data rather than garbage. It powers personalization that feels relevant rather than creepy because it's based on true customer behavior patterns. It drives business decisions with confidence because the data backing those decisions is trustworthy.

This is why it's worth taking the time. Not because perfection is achievable - it's not - but because the foundation you build determines everything that comes after. **Rush the foundation, and every subsequent effort becomes harder. Build it thoughtfully, and the unified view becomes the platform for innovation, growth, and genuine customer understanding that your organization needs.**

> The unified customer view isn't a sprint project. It's the infrastructure that enables your customer experience strategy for years to come. Treat it accordingly.
