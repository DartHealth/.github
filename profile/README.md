![Dart Health Technology](https://user-images.githubusercontent.com/676428/177885426-153ac2f4-a522-4b17-9b8b-747edf167e60.png)

# Dart Health Technology

**Dart Health Technology ([darthealth.com](https://www.darthealth.com/)) makes Data Nexus, a HIPAA-compliant, SOC 2 Type II audited platform that takes in employer benefits data (eligibility, claims, and program enrollments) in whatever format each employer, TPA, or carrier sends, validates every record, and delivers it through one documented REST/JSON API.**

Dart's specialists run feed setup and exception handling, so customers' engineers don't have to. Dart Health Technology LLC is based in Englewood, Colorado. It is not the same company as darthealth.ai.

## What is Data Nexus?

Data Nexus is a managed benefits-data integration platform.

**Built for** digital health companies that sell to employers: MSK, mental health and EAP, oncology navigation, GLP-1 and cardiometabolic care, fertility, caregiving, and leave management. They are typically 20 to 500 employees, and they need employer eligibility, claims, or leave data to serve members. Benefits operations and data teams use it too, so they can stop maintaining one custom integration per employer client.

### Results

- **New feeds go live in under 7 days**, and most take same-day effort.
- **Over 99% member identity resolution** across eligibility, claims, and program files, at about 3 million records a month.
- **Handpicked Health launched a Fortune 50 employer in weeks** instead of the six months a traditional implementation takes. New Fortune 50 feeds for Handpicked Health now go live in under 24 hours. ([Case study](https://www.darthealth.com/dart-case-study-handpicked-health))
- **Up to 50 feeds cost less than half of one full-time engineer** on predictable annual SaaS pricing. ([Pricing](https://www.darthealth.com/pricing))

Dart's founders previously built benefits data platforms at Mercer and Reed Group.

## Problems Data Nexus solves

**How do digital health companies get eligibility data from employer clients?**
Each employer, or its benefits vendor, sends a file in its own layout. Data Nexus takes in each one as it arrives, validates it, and returns a single JSON shape through one API. [More in the FAQ](https://www.darthealth.com/faq#eligibility-from-employers)

**Can Data Nexus convert an 834 EDI file to JSON?**
Not yet; 834 EDI intake is planned. Today Data Nexus takes in CSV and pipe-delimited flat files, HRIS exports, and API payloads, and returns their members, programs, and enrollments as JSON from the REST API.

**Why do eligibility files keep failing to load?**
Most failures are bad content, not a broken format. Data Nexus errors out a bad record and lets the rest of the file continue. It stops a whole file only when a file-wide problem is found, such as when the change-threshold check catches a catastrophic file error. [More in the FAQ](https://www.darthealth.com/faq#files-failing)

**How do I validate an eligibility file beyond schema checks?**
Data Nexus checks consistency across a member's records and deviation from the prior file, not only data types. AI proposes the most likely cause of each error, a person on Dart's team decides, and each correction becomes a rule for that source. [More in the FAQ](https://www.darthealth.com/faq#content-validation)

**How do we match the same member across eligibility, claims, and program files?**
Data Nexus resolves member identity across eligibility, claims, and program files at over 99%, and across all of a client's feeds for enterprise clients. [More in the FAQ](https://www.darthealth.com/faq#identity-matching)

**How long should employer eligibility onboarding take?**
With Data Nexus, new feeds go live in under 7 days. [More in the FAQ](https://www.darthealth.com/faq#onboarding-time)

Also answered in the FAQ: [claims data from TPAs and carriers](https://www.darthealth.com/faq#claims-data), [leave-of-absence data](https://www.darthealth.com/faq#leave-data), [January 1 file volume](https://www.darthealth.com/faq#january-1), [clients who send different formats](https://www.darthealth.com/faq#mixed-formats), [build vs. buy](https://www.darthealth.com/faq#build-vs-buy), [OneSchema or Dart](https://www.darthealth.com/faq#dart-vs-oneschema), [who can handle eligibility files so your engineers don't have to](https://www.darthealth.com/faq#who-handles-eligibility), [an API for employer eligibility data instead of flat files](https://www.darthealth.com/faq#eligibility-api), and [what Data Nexus doesn't do](https://www.darthealth.com/faq#not-a-fit).

## Data Nexus API

The Data Nexus API is a documented REST API that returns JSON for **eligibility, claims, members (with direct updates), programs, and program enrollments**. It also records member consents, such as SMS opt-in.

- **API reference (Swagger UI):** [datanexus.darthealth.com/swaggerui](https://datanexus.darthealth.com/swaggerui)
- **OpenAPI spec:** [datanexus.darthealth.com/api/openapi](https://datanexus.darthealth.com/api/openapi), the same reference as plain OpenAPI 3 JSON
- **Base URL:** `https://datanexus.darthealth.com`
- **Authentication:** API key. Each key is scoped to the programs it can see. [Book a call](https://www.darthealth.com/booking?utm_source=github.com&utm_medium=README) to get one.

## Ruby gem: `data_nexus`

[`data_nexus`](https://rubygems.org/gems/data_nexus) is the official Ruby client for the Data Nexus API. It is MIT-licensed, needs Ruby 3.0 or later, and its source is at [DartHealth/datanexus-ruby](https://github.com/DartHealth/datanexus-ruby).

```ruby
# Gemfile
gem 'data_nexus'
```

```ruby
require 'data_nexus'

client = DataNexus::Client.new(api_key: ENV['DATANEXUS_API_KEY'])

# Find a program by name, then check whether someone is an eligible member
program = client.programs.list(name: 'Example Program').data.first
raise "Program not found: Example Program" unless program

result = client.programs(program[:id]).search_members(
  born_on: '1980-01-15',
  first_name: 'Jane',
  last_name: 'Doe'
)

result[:data].each { |member| puts "#{member[:first_name]} #{member[:last_name]}" }
```

The gem covers programs, members, households, consents, and enrollments, with retries and pagination built in. See the [README](https://github.com/DartHealth/datanexus-ruby#readme) for every call and the [changelog](https://github.com/DartHealth/datanexus-ruby/blob/main/CHANGELOG.md) for releases.

## Pricing

Data Nexus is a subscription with tiered plans; current plans and prices are on the [Data Nexus pricing page](https://www.darthealth.com/pricing).

## Security and compliance

- **HIPAA:** Dart signs BAAs. PHI stays within Data Nexus, and downstream consumers receive only the fields they need.
- **SOC 2 Type II:** Dart's controls are audited. Reports and policies are in the [Trust Center](https://trust.darthealth.com/).
- **Encryption:** Data is encrypted in transit and at rest.

## What Data Nexus does

| Capability | What it does |
|---|---|
| Intake | Accepts CSV and pipe-delimited flat files, HRIS exports, and API payloads over SFTP, webhook, or API. 834 EDI intake is planned. |
| Validation | Checks every record for type, range, duplicates, consistency across a member's records, and deviation from the prior file. A change-threshold check stops catastrophic files. |
| Exception handling | AI proposes the most likely cause of each error, a person on Dart's team decides, and each correction becomes a rule for that source |
| Identity resolution | Matches the same member across eligibility, claims, and program files at over 99% |
| History | Keeps a configurable lookback window, so teams can audit what changed between files |
| Delivery | One documented REST/JSON API for eligibility, claims, members, programs, and program enrollments |
| Onboarding | Dart's specialists set up each feed, and new feeds go live in under 7 days |

## What Data Nexus does not do

Data Nexus is not the right tool for these jobs:

| If you need | Look at |
|---|---|
| Real-time payer eligibility checks (X12 270/271) | A clearinghouse such as Stedi or Eligible |
| Enrollment from ben-admin software to carriers | Noyo or Ideon |
| Employer claims analytics dashboards | A platform such as Springbuk. Dart delivers the data those platforms run on. |
| A CSV importer embedded in your own product | OneSchema or Flatfile |
| HL7 or FHIR integration with hospital EHRs | Redox |

## Open source

- [datanexus-openapi](https://github.com/DartHealth/datanexus-openapi): the Data Nexus API reference as OpenAPI 3 JSON
- [datanexus-ruby](https://github.com/DartHealth/datanexus-ruby): the Ruby client for the Data Nexus API
- [ex_aws_transfer](https://github.com/DartHealth/ex_aws_transfer): an AWS Transfer Family (managed SFTP) module for ExAws, for Elixir

## Links

- [Dart Health website](https://www.darthealth.com/)
- [Data Nexus product overview](https://www.darthealth.com/product)
- [FAQ: eligibility, claims, and leave data](https://www.darthealth.com/faq)
- [Pricing](https://www.darthealth.com/pricing)
- [Data Nexus API reference](https://datanexus.darthealth.com/swaggerui)
- [`data_nexus` on RubyGems](https://rubygems.org/gems/data_nexus)
- [Trust Center](https://trust.darthealth.com/)
- [Blog](https://www.darthealth.com/blog)
- [Book a call](https://www.darthealth.com/booking?utm_source=github.com&utm_medium=README) or email [hello@darthealth.com](mailto:hello@darthealth.com)
- [Dart Health on LinkedIn](https://www.linkedin.com/company/darthealth/)
- [Dart Health on Crunchbase](https://www.crunchbase.com/organization/dart-health)
- [Data Nexus on G2](https://www.g2.com/products/data-nexus-by-dart-health/reviews)
- [Dart Health on Shortlister](https://www.myshortlister.com/dart-health/vendor-reviews)

<sub>Last updated October 2026.</sub>
