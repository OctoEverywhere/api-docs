---
title: Free 3D Printing AI Failure Detection APIs
description: Bring advanced, highly accurate AI 3D print failure detection to your products, print farm, or personal projects.
og_title: Free 3D Printing AI Failure Detection APIs
og_description: Bring advanced, highly accurate AI 3D print failure detection to your products, print farm, or personal projects.
---

# 🤖 AI Print Failure Detection APIs

**Catch failed prints sooner. Save filament, time, and money.**

[Gadget](https://octoeverywhere.com/gadget?source=oe_docs_gadget_api_overview_intro) brings AI failure detection with **industry-leading accuracy** to your 3D printing workflow. Use it with compatible print farm software, add it to a personal project, or bring it to your customers under your own brand. It is the same AI that powers OctoEverywhere's print failure detection, available wherever you want to build or use an integration.

**You do not have to be a developer.** If your software already includes Gadget support, your API key connects it to our detection service. Your software handles sending images and responding to detections; you manage your key, usage, and billing on one page.

[Get Your Free API Key](https://octoeverywhere.com/gadgetapi?source=oe_docs_gadget_api_overview_hero_key){ .md-button .md-button--primary }
[See Pricing](#pricing){ .md-button }

!!! tip "500 free print hours every month. No card required."
    Every account includes **90,000 free inspections per monthly billing period** - equivalent to **500 print hours.** **A Free Usage Only limit is enabled by default.** Paid usage is optional: set up billing and explicitly turn off the limit when you are ready for more.

## Who Is It For? { #what-you-can-build }

| User | Put Gadget to Work |
| --- | --- |
| **Everyday makers** | Set up the [OctoEverywhere plugin](https://octoeverywhere.com/getstarted?source=oe_docs_gadget_api_overview_plugin_get_started) for you printer to unlock free and unlimited AI failure detection powered by Gadget. |
| **Personal projects** | Add AI failure detection to your own dashboards, apps, automations, and weekend experiments. |
| **Print farms** | Help catch failures before more filament and print time are wasted. If your farm software has Gadget built in, add your API key and follow its setup instructions. No coding required. |
| **Businesses & white-label products** | Offer AI print failure detection under your own brand, with no OctoEverywhere attribution required. Scale to thousands or millions of users without building and hosting your own detection models. [Custom volume pricing is available.](https://octoeverywhere.com/support?instant=1&source=oe_docs_gadget_api_overview_white_label) |

Gadget combines industry-leading AI image classification with a temporal AI model that follows each print over time. Together, they identify potential failures by analyzing both individual webcam images and how the print develops. Your software can use the resulting print-quality signals and recommendations to warn you or pause a print. Available alerts and automatic actions depend on the software you use.

The models are continuously improved with community feedback and run on OctoEverywhere's global server network. You get the detection service while your software handles the printing workflow.

## Get Your API Key

1. Open [OctoEverywhere.com/GadgetAPI](https://octoeverywhere.com/gadgetapi?source=oe_docs_gadget_api_overview) and sign in or create an account.
2. Choose **Copy API Key** - no credit card or billing setup is needed.
3. Paste the key into your software's Gadget or OctoEverywhere AI failure detection settings and follow its instructions to enable detection. If you are building your own integration, start with the [SDK](developer-docs/overview.md#sdks) or [API overview](developer-docs/overview.md#api-overview) in our Developer Docs.

An API key is the private code that connects your software to your Gadget account. Keep it private, and use the same account's key across your printers and tools. Their inspection usage adds together under that account's monthly allowance.


### Stay Free or Enable Paid Usage

Your [Gadget API account page](https://octoeverywhere.com/gadgetapi?source=oe_docs_gadget_api_overview_account_usage) shows your key, current usage, and billing-period dates. **Free Usage Only is enabled by default**, so inspection requests stop when you reach your free allowance. The allowance renews each monthly billing period; your key stays the same.

- **Want to stay free?** Leave the limit on. You can add billing details in advance and still keep the free-only limit enabled.
- **Need more inspections?** Select **Set Up Billing**, add a payment method, then turn off **Free Usage Only** to allow pay-per-call usage. You can re-enable the Free Usage Only limit whenever you wish.

When inspections stop at the limit, your software receives `OE_FREE_USAGE_LIMIT_REACHED`. See [Error Handling](developer-docs/overview.md#error-handling) for more details.

## Pricing

**Your first 500 print hours each month are free.** If you choose to enable paid usage, you only pay for inspection calls beyond your free allowance, with lower rates as your monthly usage grows. Prices below are shown in USD. All accounts have an enabled-by-default free usage limit stop, that can be turned off at anytime.

### Pricing by Print Hours

All print-hour pricing on this page assumes **one inspection every 20 seconds**: 180 inspection calls per print hour. A print hour is one printer monitored for one hour; usage across printers adds together. You can see your current usage on your [Gadget API account page.](https://octoeverywhere.com/gadgetapi?source=oe_docs_gadget_api_pricing)

| Monthly Print Hours | Price per 1,000 Print Hours |
| ------------------- | -------------------------- |
| First 500           | Free                       |
| Over 500 to 5,000    | $5.40                      |
| Over 5,000 to 25,000 | $4.50                      |
| Over 25,000         | $3.60                      |

### Pricing by Inspection Calls

Billing is based on successful [Process API](process.md) inspection calls. [Create Context](create-context.md) calls are free. The call rates below are the same pricing as the print-hour table, expressed per 1,000 calls.

| Monthly Inspection Calls | Price per 1,000 Calls |
| ------------------------ | -------------------- |
| First 90,000              | Free                 |
| Over 90,000 to 900,000     | $0.030               |
| Over 900,000 to 4,500,000  | $0.025               |
| Over 4,500,000            | $0.020               |

These are **progressive tiers**, based on total monthly usage across your account: each rate applies only to usage within that tier. Reaching a new tier does not change the price of earlier calls, and you do not need to buy blocks of 1,000 hours or calls.

For example, **6,000 print hours in a month cost $28.80** at the 20-second interval: the first 500 hours are free, the next 4,500 cost $24.30, and the remaining 1,000 cost $4.50.

Use **`NextProcessIntervalSec.Recommended` as the default inspection interval**, taking the value from the latest Process API response. You can inspect faster than recommended, provided you respect `NextProcessIntervalSec.Minimum`, currently 5 seconds. Shorter intervals use more calls, consume the free allowance sooner, and increase paid usage costs; longer intervals use fewer calls. The free allowance and billing are measured in calls, so the print hours covered by each tier change when you use a different interval.

We offer volume pricing for customers with high API demand. [Contact us to discuss details.](https://octoeverywhere.com/support?source=oe_docs_gadget_api_overview_volume_pricing&instant=1)

[Get Your Free API Key](https://octoeverywhere.com/gadgetapi?source=oe_docs_gadget_api_overview_pricing_key){ .md-button .md-button--primary }

## Privacy

Images submitted through the public Gadget APIs are **not used for training AI models**.

## Developer Docs

Building your own integration? Explore our SDKs, API workflow, error handling, and high-availability guidance.

[Read the Developer Docs](developer-docs/overview.md){ .md-button .md-button--primary }

## Get in Touch

Need help connecting your software, choosing the right setup for your farm, or building an integration? We would love to help you get started.

[Contact Our Team](https://octoeverywhere.com/support?instant=t&source=oe_docs_gadget_api_overview_contact&instant=1){ .md-button .md-button--primary }
