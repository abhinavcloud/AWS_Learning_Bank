# LP - Invent and Simplify

# Story - Migrating from high cost iText pdf generator to lower and customizable pdfReactor html to pdf tool.


## Situation:

>As part of Telstra's DaVinci Program — building an AWS cloud digital stack for Telstra's Fintech systems — I was part of the Invoicing domain. Our bill presentment system relied on a third-party tool, iText, to convert billing JSON into PDF documents. In late 2024, iText flagged that continuing to use the tool on our AWS platform would require a new licensing/support fee of roughly half a million dollars — a cost the group couldn't absorb, and one that was flagged as a high-risk third-party dependency.

## Task:

>I proposed we evaluate build-vs-buy alternatives rather than accept the licensing hit, and led the technical evaluation of how we could remove the risk.

## Action:

>I designed a two-part approach to replace iText. First, I broke each PDF template — invoices, receipts, credit notes, adjustment notes — into static sections (built as HTML/CSS) and dynamic, parametrized sections. I then built an in-house overlay tool that took the billing JSON and mapped its data into the parametrized sections to generate a unique HTML document per customer, which we then rendered to PDF using PDF Reactor — a lower-cost HTML-to-PDF engine — with all styling and formatting intact.

>Once this was working for invoicing, I saw it wasn't actually invoicing-specific — the same static/dynamic template pattern could generate any PDF from any JSON input. I proposed generalizing it into a standalone shared service, Document Manager, rather than leaving it as a one-off fix inside the Invoicing domain.

## Result:

>This avoided the roughly $500K/year iText licensing fee that had been flagged as a group-level risk, replacing it with a substantially lower-cost tool — I don't have the exact net figure, but it removed the immediate cost and dependency risk entirely. More importantly, Document Manager became a shared platform capability — instead of each domain building its own PDF generation, Payments and Journals now use it too, for things like collection/debt letters, adjustment notices, and credit notes. What started as a cost-avoidance fix became reusable infrastructure across the business.





