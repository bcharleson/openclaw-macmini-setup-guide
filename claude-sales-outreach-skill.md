# Sales Outreach - Claude Skill

## Description

Generate personalized, high-converting sales outreach messages for email, LinkedIn, and other channels. This skill helps sales teams craft compelling messages that resonate with prospects based on their company, role, industry, and pain points.

## When to Use This Skill

Invoke this skill when users ask about:
- Writing cold outreach emails
- Creating LinkedIn connection requests or InMails
- Personalizing sales messages at scale
- Follow-up message sequences
- Re-engagement campaigns
- Multi-channel outreach strategies
- A/B testing different message variations

## Prerequisites

Before generating outreach, gather:
- **Prospect Information**: Name, company, title, industry
- **Research Context**: Recent news, funding, job changes, mutual connections
- **Your Value Proposition**: What problem you solve, for whom, and how
- **Call to Action**: What you want them to do (book a meeting, reply, etc.)
- **Tone Preference**: Professional, casual, direct, consultative

## Instructions

### 1. Gather Prospect Intelligence

Ask the user for:
- Prospect's name and title
- Company name and industry
- Recent company news (funding, expansion, product launches)
- Trigger events (new role, company growth, pain point signals)
- Mutual connections or commonalities
- Social media activity or content they've shared

### 2. Understand the Offer

Clarify:
- What product/service are you selling?
- What's the core value proposition?
- What pain point does it solve?
- Who is your ideal customer profile (ICP)?
- What differentiates you from competitors?
- What's the desired outcome of this message?

### 3. Choose Outreach Channel

Determine the channel:
- **Cold Email**: More detailed, can include links/attachments
- **LinkedIn Connection Request**: 300 character limit, must be compelling
- **LinkedIn InMail**: Similar to email but professional context
- **Twitter/X DM**: Brief, conversational, social-first
- **Follow-up Sequences**: Multi-touch across channels

### 4. Generate Personalized Message

Create a message that:
- **Opens with relevance**: Reference something specific about them/their company
- **Demonstrates value quickly**: Show you understand their challenges
- **Makes it about them**: Focus on their goals, not your product
- **Has a clear CTA**: Make it easy to say yes to a small next step
- **Keeps it concise**: Respect their time (< 150 words for email, < 100 for LinkedIn)

### 5. Apply Best Practices

Ensure messages follow proven frameworks:

**AIDA Framework:**
- Attention: Hook with personalized insight
- Interest: Connect to their pain point
- Desire: Show value/outcome
- Action: Clear, low-friction CTA

**PAS Framework:**
- Problem: Identify their challenge
- Agitate: Amplify the pain point
- Solve: Position your solution

**BAB Framework:**
- Before: Current state with pain
- After: Desired state with solution
- Bridge: How you help them get there

### 6. Optimize for Deliverability (Email)

For cold emails:
- Avoid spam trigger words (free, guarantee, limited time, act now)
- Use plain text or minimal formatting
- Include a clear unsubscribe option
- Personalize beyond first name (company, role, industry)
- Keep subject lines under 50 characters
- Avoid excessive links (max 1-2)
- Test for spam score

### 7. Create Follow-Up Sequence

Design a multi-touch sequence:
- **Touch 1**: Initial value-driven outreach
- **Touch 2** (3 days later): Add social proof or case study
- **Touch 3** (4 days later): Different angle or resource
- **Touch 4** (5 days later): Break-up email or final attempt
- **Cross-channel**: Mix email + LinkedIn + phone

### 8. A/B Test Variations

Generate multiple versions testing:
- Different subject lines
- Opening hooks
- Value propositions
- CTAs
- Length (short vs medium)
- Tone (formal vs casual)

## Output Format

Provide:
1. **Subject Line** (for email) or **Opening** (for LinkedIn)
2. **Message Body** (formatted for the chosen channel)
3. **Rationale**: Why this approach works for this prospect
4. **Variations**: 2-3 alternative versions
5. **Follow-up Suggestions**: Next touches in the sequence

## Example Usage

**User:** "Help me write a cold email to the VP of Sales at a SaaS company that just raised Series B funding."

**Agent:** "Great! Let me help you craft a compelling outreach message. First, let me gather some details:

1. What's the prospect's name and company?
2. What does your product/service do?
3. What specific pain point does it solve for VPs of Sales?
4. Do you have any relevant case studies or social proof?
5. What's the goal of this email? (e.g., book a 15-min discovery call)

Once I have these details, I'll generate:
- A personalized subject line that ties to their funding news
- An email body using the PAS framework
- 2-3 variations for A/B testing
- A follow-up sequence for if they don't respond

Let's make this outreach convert!"

## Best Practices

### Subject Lines That Work

**Pattern 1: Question**
- "Quick question about [company]'s sales enablement?"
- "Thoughts on scaling [specific process]?"

**Pattern 2: Mutual Connection**
- "[Name] suggested I reach out"
- "Following up on [Name]'s introduction"

**Pattern 3: Personalized Insight**
- "Congrats on [recent news]"
- "Saw your post about [topic]"

**Pattern 4: Value-First**
- "[Specific outcome] for [company]"
- "Resource: [Topic] for [industry] teams"

### LinkedIn Connection Request (300 chars)

**Template:**
```
Hi [Name], noticed [specific detail about them/company]. I work with [similar companies/roles] on [specific outcome]. Would value connecting and sharing insights on [relevant topic]. No sales pitch—just genuinely interested in your take on [industry trend].
```

### Cold Email Template

**Template:**
```
Subject: [Personalized hook]

Hi [Name],

[Personalized opening - show you did research]

[One-sentence value prop tied to their pain]

[Social proof - who else you've helped]

[Clear, low-friction CTA]

Best,
[Your name]
```

### Follow-Up Email Template

**Template:**
```
Subject: Re: [Original subject]

Hi [Name],

Following up on my note from [day].

[New angle or resource]

Still interested in [specific outcome]?

[Even easier CTA than before]

Best,
[Your name]
```

## Metrics to Track

Help users measure success:
- **Open rate**: Subject line effectiveness (target: 30-40%)
- **Reply rate**: Message quality (target: 5-15% for cold, 30%+ for warm)
- **Meeting booked rate**: CTA conversion (target: 2-5%)
- **Unsubscribe rate**: Relevance check (keep under 1%)

## Common Mistakes to Avoid

❌ **Generic templates** - "I hope this email finds you well"
❌ **Talking only about your product** - Make it about them
❌ **Too long** - Respect their time
❌ **Vague CTA** - "Let me know if you're interested"
❌ **No personalization** - At minimum: name, company, role
❌ **Spam trigger words** - "Free", "guarantee", "limited time"
❌ **Too many asks** - One clear next step only

## Advanced Techniques

### Trigger-Based Outreach

Reach out when prospects show intent signals:
- Job changes
- Funding announcements
- Product launches
- Hiring for relevant roles
- Content engagement
- Website visits (if tracked)

### Account-Based Sequences

For strategic accounts:
- Research multiple stakeholders
- Personalize at account level, not just person level
- Multi-thread with different roles
- Coordinate touches across your team
- Provide value before asking

### Video Messages

Stand out with personalized video:
- Record 30-60 second Loom/Vidyard
- Address them by name
- Show specific research
- Explain value visually
- Include in email as alternative

## Integration Tips

This skill works well with:
- **CRM systems**: Salesforce, HubSpot, Pipedrive
- **Sales engagement tools**: Outreach, SalesLoft, Apollo
- **LinkedIn Sales Navigator**: For prospect research
- **Email verification**: Hunter, NeverBounce, ZeroBounce
- **AI research tools**: Clay, Amplemarket, 6sense

## Files

- `claude-sales-outreach-skill.md`: This sales outreach skill
- `claude-skill.md`: OpenClaw setup assistance skill
- `claude-upgrade-skill.md`: OpenClaw upgrade skill

## Repository

https://github.com/bcharleson/openclaw-macmini-setup-guide
