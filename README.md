# SQL-project
A number of SQL queries

## ER diagram description
**acquisition**
Contains information about the purchases of one company by another. The table includes the following fields:
- Primary key id: Identifier or unique purchase number.
- Foreign key acquiring_company_id: References the company table, identifying the acquiring company, the one buying another company.
- Foreign key acquired_company_id: References the company table, identifying the company being acquired.
- term_code: Payment method for the transaction: cash, stock, or cash and stock (a combination of cash and stocks).
- price_amount: The purchase amount in dollars.
- acquired_at: Date of the transaction.
- created_at: Date and time of record creation in the table.
- updated_at: Date and time of record update in the table.

**company**
Contains information about startup companies. The table includes the following fields:
- Primary key id: Identifier or unique company number.
- name: Company name.
- category_code: Company's field of activity, e.g., news or social.
- status: Company status (acquired, operating, IPO, closed).
- founded_at: Date of the company's founding.
- closed_at: Date of closure if the company no longer exists.
- domain: Company website domain.
- twitter_username: Company's Twitter profile name.
- country_code: Country code (e.g., USA for the United States, GBR for the United Kingdom).
- investment_rounds: Number of rounds in which the company participated as an investor.
- funding_rounds: Number of rounds in which the company attracted investments.
- funding_total: Total amount of investments raised in dollars.
- milestones: The number of significant milestones in the company's history.
- created_at: Date and time of record creation in the table.
- updated_at: Date and time of record update in the table.

**education**
Stores information about the education levels of company employees. The table includes the following fields:
- Primary key id: Unique record number for education information.
- Foreign key person_id: References the people table, identifying the person whose information is in the record.
- degree_type: Educational degree type, e.g., BA (Bachelor of Arts) or MS (Master of Science).
- institution: Name of the educational institution or university.
- graduated_at: Date of graduation.
- created_at: Date and time of record creation in the table.
- updated_at: Date and time of record update in the table.

**fund**
Contains information about venture capital funds. The table includes the following fields:
- Primary key id: Unique fund number.
- name: Fund name.
- founded_at: Fund's founding date.
- domain: Fund's website domain.
- twitter_username: Fund's Twitter profile name.
- country_code: Country code for the fund.
- investment_rounds: Number of investment rounds in which the fund participated.
- invested_companies: Number of companies in which the fund has invested.
- milestones: The number of significant milestones in the fund's history.
- created_at: Date and time of record creation in the table.
- updated_at: Date and time of record update in the table.

**funding_round**
Contains information about investment rounds. The table includes the following fields:
- Primary key id: Unique investment round number.
- Foreign key company_id: References the company table, identifying the company participating in the investment round.
- funded_at: Date of the investment round.
- funding_round_type: Type of investment round (e.g., venture, angel, series A).
- raised_amount: The amount of investment raised by the company in this round in dollars.
- pre_money_valuation: Pre-investment valuation of the company in dollars.
- participants: Number of participants in the investment round.
- is_first_round: Indicates whether this round is the company's first.
- is_last_round: Indicates whether this round is the company's last.
- created_at: Date and time of record creation in the table.
- updated_at: Date and time of record update in the table.

**investment**
Contains information about investments by venture capital funds in startup companies. The table includes the following fields:
- Primary key id: Unique investment number.
- Foreign key funding_round_id: References the funding_round table, identifying the investment round.
- Foreign key company_id: References the company table, identifying the startup company receiving the investment.
- Foreign key fund_id: References the fund table, identifying the fund making the investment.
- created_at: Date and time of record creation in the table.
- updated_at: Date and time of record update in the table.

**people**
Contains information about employees of startup companies. The table includes the following fields:
- Primary key id: Unique employee number.
- first_name: Employee's first name.
- last_name: Employee's last name.
- Foreign key company_id: References the company table, identifying the startup company where the employee works.
- twitter_username: Employee's Twitter profile name.
- created_at: Date and time of record creation in the table.
- updated_at: Date and time of record update in the table.
