# Record Status used in many tables #

Value | Meaning
-----:| -------
0     | Active record
1     | Added record
2     | Updated record
3     | Deleted record

When a record is deleted, the active record's status is changed to 3 (deleted), and any existing status 1 or 2 record is deleted. It seems that at any given time there can only be one status 0 record plus one record that is either added, updated, or deleted status? If updates are made when a status 1 or 2 record already exsists, there is no record made showing values before the update. This observation is based on the logic I see in program E90T1901.

# Table and Column Documentation #

Documentation for each table and column follows. A lot of this documentation was copied from the TRACS-EU Royalty System User Manual Version 1.1, last modified July 20, 2012.

## Company ##
Table which holds information about each company within a site. See [Company.Company]() for more information about what a _processing company_ is. This table also holds information for _reporting companies_, _contractual partner companies_, etc.

### Company.Company ###
The _Processing Company_ is the principle element on the concept of the TRACS-EU system: it is mandatory in many files and tables and for batch production and batch reporting.

All data files are held under a key of Processing Company. Since this element is used so often in the database, the column name used is simply _Company_ rather than _ProcessingCompany_. If it�s not the processing company that is being referred to, an adjective will be used, such as _ReportingCo_, _ContractualCo_, or _AdminCo_.

You will have access to at least one processing company (as specified in The User ID Table 91); almost all sites contain data for more than one company.

A **Processing Company** represents an entity in the local organisation (a.k.a. site) for which separate processing and/or reporting is required_.

All data (articles, contracts, recordings etc.) is stored separately by processing company. Codes data (e.g. territory codes, sales transaction types) can also be stored per Processing Company. It is however possible for codes data to be held under the default Processing Company number of 0000000. If no individual codes table data exists for a specific Processing Company, then batch & on-line activities will look for data under company zero. This saves time & effort, since many codes tables are valid for all companies in the same site).

The same article number may exist under more than one processing company for the same site; this is normal, since often articles contain the repertoire of more than one local company. Sales can be transferred from one company to another on the same DB using the _Intercompany_ function. In the TRACS-EU Royalty System User Manual see Article File (participation) & the related Batch Processing step.

## Article ##
TRACS-EU considers an _article_ to be either a physical product (a CD, for example) or a product with
a key (UPC number etc.) in the same format as a physical article (e-articles, for example).

An _article_ is also known as a _product_.

_Recordings_ (mostly under key of the ISRC) are held in a separate table: the Recording table (sales batches
� and all further processing � are separated between those with Article Numbers as key in the sales
line and those with Recording Numbers as key).

### Article.Company ###
Processing company. See [Company.Company]() for details.

### Article.MainCodeGroup ###
Type | Meaning
----:| -------
0    | GCS (old numbering system; few articles still use this)
1    | PPC/UPC (an all-numeric Universal number)
9    | Other (includes 3rd party numbers). Causes [Article.CatAdminNo]() to be a required field.

This field should not be confused with [Recording.CodeGroup](). Any time you see the word _Main_ in front of _CodeGroup_, it is referring to the article code group and not the recording code group.

### Article.ArticleNo ###
The three fields [Article.MainCodeGroup](), this field [Article.ArticleNo](), and [Article.CatAdminNo]() combined together uniquely identify an article (a sound or video carrier such as CD, single, VHS video, etc.). These three fields are not required in the Royalty table, since it's possible we can receive sales for a new article before it's set up in the [Article]() table.

This field [Article.ArticleNo]() can be broken down into three smaller parts, but this is rarely done so it was decided to have this article number be one 13 character column in the database rather than three smaller columns. The three sub-fields are:

1. **Prefix** (alphanumeric 5 characters)
1. **Catalog Number** (numeric 7 digits), also known as the _suffix_, and not to be confused with the Catalog _Administrator_ Number, which is a separate column, see [Article.CatAdminNo]().
1. **Code** (alphanumeric 1 character), which is actually the last character of the article number even though the previous sub-field is often called the _suffix_.

The [ArticleStructure]() table can help to define the format of article numbering within the system and to provide some further checks. If this table includes limitations on the format of article numbers, input in the article file must conform to these limitations. There is only one general rule: if [Article.MainCodeGroup]() (MCG) = 9, [Article.CatAdminNo]() is a mandatory field (this is a Universal article format rule), and must exist in table [Company](). Therefore, this Catalog Administrator number is actually a company number. Therefore, the proper name would be CatAdminCoNo, but the term CatAdminNo is so commonly used that it would be confusing to call it anything else.

If the [ArticleStructure]() table is set up according to Universal Music standards, it is possible to allow the system (especially for MCG = 1 articles) to generate the prefix (5 characters) portion of the article number. If the user inputs MCG = 1 and a 7 digit catalog number (a.k.a. the suffix), the system will create the prefix part. See documentation on the [ArticleStructure]() table for the method of set-up to achieve this if required (note: this table is rarely used now).

### Article.ArtistPriceLevel ###
If the user wants to define an article only to be accounted by all associated contracts at a specific Price Level (high, medium, budget etc.), then a code from the [PriceLevel]() table should be input here. N is the default value which should be defined in the table with a description of _Normal_.

In batch processing, the system will then search for a subdivision (i.e. the royalty rate to be paid - see [Subcontract]() and [Subdivision]()) using the price level as part of the search key (and not the top price percentage). Most TRACS-EU sites use this facility only exceptionally where they want to force a particular price level for accounting, regardless of the actual selling price of the article; the value used here by most sites for most articles is N (normal); in this case, the subdivision search is made using the top price percentage as part of the subdivision key, not the price level.

Values from Germany database used during MonthlyPaymentsProject UAT testing:

Value | Count
-----:| ------:
blank | 65
B     | 1160
C     | 36561
H     | 2321
M     | 133
N     | 231266

### Article.ArtistRoyaltyStatus ###
This defines the route to be taken in batch processing when sales of the article confront the system. An article can have contracts (the 'normal' method) and/or tracks linked to it, or indeed both contracts and tracks. A track is usually one recording within an article (e.g. a song on an album).

The values allowed here are:

Value | Meaning
-----:| -------
N     | No contracts attached (or if any, they are ignored). There is a Validation Warning Report for sales processed on such articles
C     | Processing will only look for any Contracts attached
R     | Processing will only look for any Tracks attached
X     | Processing will look for **both** Contracts & Tracks attached
I     | Incomplete record: batch process will not calculate royalties. The sales line goes into error, and is reported as such on the sales validation report.

In batch processing, a _sale_ is the initialising factor. Sales of an Article Number are confronted with that Article file record.

The above values C, R, and X will define whether to create the (royalty?) data directly for the Contracts attached to the Article, or create the contracts indirectly via the [Recording]() table, or to create the Contract data via both methods at the same time - this is a very important field.

See the [Article_Link_Subcontract]() table for more information.

### Article.ReleaseDate ###
This field is used in reserve processing (q.v.) in order to define the interval between the release date of the article and the sales date of the sales line (and from there to decide if and how many reserves are to be taken). It is not otherwise used.

### Article.ConfigurationCode ###
Any input in this field must be in the [Configuration]() table; the value will be stored in processing and used as (part of) a key:
* to create a price uplift (e.g. dealer to retail) if necessary
* to establish the top price of the sales line (if necessary)
* to define the reserve pattern at contract level
* to define escalation scaling parameters
* to select the correct subdivision (= royalty rate) **(especially critical)**

The Configuration code value is used to determine if the article is a CD, cassette tape, 8-track tape, LP album, DVD, VHS video tape, Blu-ray disc, interenet download, etc.

### Article.EquivalentConfigurationCode ###
This field is not as important as the main configuration field [Article.ConfigurationCode](), however any input value must exist in the [Configuration]() table. The value will be used to create a price uplift if necessary.

### Article.PackagingCode ###
Any input in this field must be in the [PackagingCode]() table; the value will be stored in processing and used as (part of) a key to select the correct subdivision (= royalty rate). Some examples are Normal, Luxury sleeve, Jukebox single, CD with extra singles, etc.

### Article.OrigArticleNo ###
This is a free format field. An enquiry is possible on this field by selecting PF10 from the main article screen.

### Article.MusicClass ###
Any input in this field must be in the [MusicClass]() table; the value will be stored in processing and used as (part of) a key to establish the top price of a sales line and a price uplift factor if necessary. Some example values are Classical, Popular, Spoken word, etc.

### Article.FirstRelease ###
Values = Y / N.

If Y, and a subcontract also contains Y, then only a price level = H subdivision can be selected, i.e. the article will only be accounted at top royalty rate. However, if the [Article.ArtistPriceLevel]() is set to anything except 'N', it over-rides this procedure.

### Article.ArtistReservePriority ###
Values = N (normal) or P (priority) -- default is N

Batch processing stores the reserve code from the article record for use in reserve processing, but over-writes it with a reserve code value from the subcontract record (if filled).

If the value = 'P', it cannot be over-written by the subcontract value (unless subcontract says NO in general to reserves).

### Article.ArtistReserveCode ###
This field is mandatory if the [Article.ArtistReservePriority]() is P, otherwise optional. Any input in this field must be in the [ReserveAutomatic]() table; the value will be stored in processing and used during the reserve process (subject to it being over-written by a subcontract parameter).

### Article.Label ###
Any value can be put here, there is no table defining label codes. Currently this field is not in use.

### Article.ProductType ###
This field is not in use.

### Article.ProjectRefNo ###
This is a free-format optional field, a value input here will be reported out to the AIF Centre as part of the AIF transmission process.

A value can also be filled automatically if the article is created as part of the process to load Sales and (new) articles from an Incoming AIF sales file (and the incoming AIF sales line contains a project reference number).

### Article.SetContentsCount ###
Number of units in the set. For example, a 3 CD Set will have a value of 3.

### Article.CopyrightStatus ###
Processing Route for Copyright:

Value  | Meaning
------:| -------
N      | No Copyright Payable
R or T | Copyright processing will only look for tracks attached (the �long route�)
S      | Short route: processing will only look to the publishers (see [Article_Array_Publisher]())
I      | Incomplete record: Process will set the record to error
X      | Both long and short routes!

### Article.CopyrightReserveCodeNormal ###
Code table value from table [ReservePattern](). The value will be stored in processing and used during the reserve process, but may be overwritten by a value in the statutory rate table.

### Article.TotalProtectedPct ###
Publisher total protected percentage. Information only -- not used in processing.

### Article.SocietyApproved ###
This flag is for information only and can be filled to indicate when the local or international copyright society has approved or finalised the share of the product which it wants to claim as copyright-payable.

### Article.CopyrightFreeStartBalance ###
In some territories, a quantity of (usually promotional) sales are allowed to be deducted for copyright; these sales are not reported. TRACS-EU works in such a way that the article file can be populated with an allowed start balance (as defined in the local copyright agreement), and this balance is then gradually reduced to zero by deducting allowed (promotional) sales. So the start balance for a new release should be input here.

The quantity of the start balance per configuration may be stored in the configuration table and automatically updated for new articles created using a local article interface programme. If the start balance is blank, this means that either no balance was input (and therefore nothing will be deducted), or the balance has all been used up already in a prior processing period. This balance is reset to the value of the [Article.CopyrightFreeUpdatedBalance]() during the Copyright File Cleaning Process.

### Article.CopyrightFreeUpdatedBalance ###
This is the copyright-free balance as updated by the TRACS-EU batch process. At the end of the processing cycle, file cleaning will move this balance to the [Article.CopyrightFreeStartBalance]() for the new processing period. If this balance is blank, this means that either no start balance was input (and therefore nothing was deducted), or the balance has all been used up already in a prior processing period.

### Article.GenericArticleNo ###
The copyright-free quantity allowed to be deducted may be specified on an article-by-article basis, or on a (generic) product basis; in this latter case, the number of the generic article - i.e. the one from which the copyright balance will be deducted is placed here (a value input here means that the balance will be deducted from the specified generic article and not the _key_ article).

Note that the generic article must exist in its own right as an article in the TRACS-EU [Article]() table.

## Article_Array_Publisher ##
A list of publishers for a copyrighted article. The Natural/Adabas version of TRACS-EU supports up to 5 publishers per article. There is no limit in the DotNet version of TRACS-EU.

* If [Article.CopyrightStatus]() = S or X there must be at least one publisher for the article in this child table.
* If [Article.CopyrightStatus]() = T, R, or N then the article doesn't need any publishers in this child table.

For any row, TRACS-EU will check that at least one statutory rate exists in the table for this publisher.

### Article_Array_Publisher.PublisherNo ###
Publisher Number of a 'one-stop' publisher. Publisher number must be in the [NameAddress]() table.

### Article_Array_Publisher.SharePct ###
Percentage of Article payable to this Publisher. Must be in the range of 0.00% to 100.00%

### Article_Array_Publisher.TotalProtectedPct ###
Information only -- not used in processing.

## Article_Link_Subcontract ##
Within TRACS-EU rows in this table are maintained on a screen that is called the Product Participation List, since this is where the contracts/subcontracts define the participation percentages that are to be paid to the parties related to the Article.

An article can pay royalties for many contracts/subcontracts, and a contract/subcontract can cover many articles. This table, therefore, resolves a many-to-many relationship, which is why _link_ surrounded by underscores is in the middle of the table name.

Rows in this table detail the contracts/subcontracts which will be accounted (i.e. paid) for sales of the article. It is possible to account (pay) contracts using only rows in this table, or using only rows in the [Article_Link_Recording]() table (a.k.a. Track table), or to use both.

The [Article.ArtistRoyaltyStatus]() should be set to:
* C to process via the contract list only (the contract list is rows in this table)
* X to process both the tracks of the article and the contracts in this list
* Either T or R if the track list only is used (the track list is rows in table [Article_Link_Recording]()). If the track list is used, the [Recording]() table entries for each of the tracks must contain in their own right a link to the contracts to be accounted (paid).

## Article_Link_Subcontract.ParticipationPct ##
The percentage of the article on which the artist performs.

## Article_Link_Subcontract.ContractualCo ##
This is often the same as the value in [Article_Link_Subcontract.Company](). But if another company number is specified, then the system will try to make an intercompany transfer of the data to the company number specified here (national & export sales only) � see Batch Processing for further information.

## Article_Link_Subcontract.ContractDescription ##
This field is displayed on screens with the label of _Additional Information_. It is used locally only: it may be printed on the royalty statement.

In the German database this field is blank about 75% of the time. In the 25% of rows that do have a value, the value is usually the same as [Contract.Description]() plus some additional text appended to the end.

## Article_Link_Recording ##
Recordings exist in their own right. e.g. when they are broadcast. When they are connected to an Article, a sequence is created, and they become track 1, track 2 etc. of the article. Therefore, this table is often referred to as the _Track_ table. Since an Article usually has many recordings, and a recording can be on several articles, this table resolves the many-to-many relationship between the two. Since this database has a standard that always gives such tables a name in the pattern of Table1_Link_Table2, it was decided to not simply name this table _Track_.

The tracks can be used for both Royalty and Copyright Processing. Rows in this table are the same for both royalty and copyright.

#### **Royalty Processing** ####
In order to use the tracks for Royalty processing, the [Article.ArtistRoyaltyStatus]() should be set to R or T (to process tracks only) or X (to process both the tracks of the article and the contracts in the article contract list). If the tracks are to be used to process royalties, the recording file entries for each of the tracks mentioned here should contain details of the contracts payable.

#### **Copyright Processing** ####
In order to use the tracks for copyright processing, the [Article.CopyrightStatus]() should be set to R or T (to process tracks only) or X (to process both the tracks of the article and the _short route_ copyright publisher(s)). If the tracks are to be used to process copyright, the recording file entries for each of the tracks must contain song information (and the relevant song(s) must contain publisher information).

### Article_Link_Recording.TrackTimeMinutes ###
On the screen where tracks are entered, the value of this field is defaulted to be a copy of the value in [Recording.DurationMinutes]().

### Article_Link_Recording.TrackTitle ###
On the screen where tracks are entered, the value of this field is defaulted to be a copy of the value in [Recording.Description]().

## ArticleCampaign ##
_As of 2010, this table is only used by the Netherlands/Belgium site and cannot be automatically used by any other site: if you see a need to use the functionality of this table, contact the TRACS-EU Help Desk._

This tables allows you to store certain parameters relating to copyright processing of products sold via a TV or radio campaign.

If sales fall within the date parameters found on rows in this table:
* The sales line price can be overridden by the specific price found in [ArticleCampaign.Price]()
* A quantity of sales can be deducted (the quantity reported for copyright is reduced by the [ArticleCampaign.MaxQty](): that deduction is removed from the system during copyright batch processing)
* For The Netherlands, if a radio campaign is included, an extra 750 sales can be deducted
* This information has to be in place before the sales are loaded (sales are usually loaded quarterly); if a sales line contains data which fills these campaign parameters, that sales line is placed (with others which also fulfill the same parameters) in a special separate sales batch (so that the lines can be changed easily, for example sales channel switch)

## ArticleCampaign.Price ##
The sales line price can be overridden by this price.

## ArticleCampaign.MaxQty ##
This quantity of sales can be deducted (the quantity reported for copyright is reduced by this amount: the deduction is removed from the system during copyright batch processing)

For The Netherlands, if a radio campaign is included, an extra 750 sales can be deducted.

## ArticleThirdParty ##
_As of 2010, this functionality is only used by the UK site._

This table is used:

1. To recode a reported article number (normally not existing in its own right in the article file) to a processing (i.e. existing) article number.
1. To make a calculation of the income received, e.g. from a third party, using an income-type contract which duplicates the conditions under which the third party accounts to you.

## ArticleThirdParty.ReportingCo ##
Third Party Company reporting the Article.

## ArticleThirdParty_Array_Contract ##
The Contract and Subcontract numbers must be in the [Contract]() and [Subcontract]() tables. The contract type must be an _income type_ in accordance with table [ContractType]().

The Third Party Article functionality can be used just to re-code article numbers, in which case no rows need to be in this child table. Any calculations made according to the data here do not appear on liability reports produced by the system. However, for validation etc., they are treated as any other type of contract. All this functionality is available, but rarely used outside of the UK site.

## ArticleStatistics ##
This data is not in use any more (it was formerly used for the old Classics Recording Cost Recovery
System), and should not be seen as a true representation of transactions processed through TRACS-EU.

Although no longer used, in fact the statistics are updated every processing period (quarter) during the
file cleaning process, so that when you view current data during the processing of a quarter, it is not yet
updated to these figures. (The program that updates article statistics was removed from the file cleaning process in 2019 as each site was converted to monthly payment processing.)

## Contract ##
_Note that contracts are not used for the processing of copyright sales._

All processing takes place by Contract and Subcontract; this is the key to the [Royalty]() table, in which all processed data is stored, and from which local Royalty Statements may be printed.


Some of the columns in this table exist at [Subcontract]() level also and the contract value is overruled by the subcontract value if it exists.

### Contract.Contract ###
According to own local (site specific) numbering system.

### Contract.Description ###
It is recommended that the Contract Description field is populated in a format so that Contracts can be easily identifiable on the Contract Selection List screen. If using individual names, it is a good idea to put the Last Name of an Artist first.

### Contract.ContractType ###
The value here must be in the [ContractType]() table.

There are several categories of Contract Types:

Category     | Description
------------ | -----------
Artist       | Majority of contracts are this type. These are always processed, and appear on liability reports.
All-In-Fee   | These contract types are the only ones selected for reporting to All-In-Fee. Refer to [Contract.ExchangeTape]() and [Contract.IntlRept](); these types also appear on liability reports.
Intercompany | ?
Override     | Intercompany contracts and Override contracts are only processed for National Sales. Refer to _Batch Processing of Intercompany_ in the TRACS-EU Royalty System User Manual for details.
Income       | Only used with Third Party Articles, does not appear on liability reports.
Pressing Fee | These contracts are not used for copyright processing. Only used with Third Party Articles, does not appear on liability reports.

### Contract.ExchangeTape ###
The Exchange Tape is only used for All-In-Fee type contracts (but is mandatory).

A sequence number here will specify the order in which AIF files are sent to the AIF Centre by all the processing companies within the same database. Unless the contract is an AIF-type, put 0 (zero) here. The [Contract.IntlRept]() flag is linked with the contract type also: populate with Y if AIF, N otherwise.

* 0 = National
* 1 = International

This value gets copied to [Royalty.ExchangeTape] during validation.

If this value in the royalty row is blank or zero, a royalty row will be included in the Earnings to GL interface calculations provided other selection criteria are met. Otherwise, [CompanyOptions.RoyaltyCalculationFlag] must be Y to include the royalty row in the calculations.

### Contract.IntlRept ###
Valid values are Y / N. Populated with Y if [Contract.IntlRept]() is AIF, N otherwise.

### Contract.ContractualCo ###
Any non-zero value here must exist on the [Company]() table and can either be the same as the processing company or a different code.

This field has no processing effect, but can be used for TRACS-EU Reports or external sorts on contract/statement information.

### Contract.EffectiveDate ###
Information only: Date on which rights begin.

### Contract.ExpirationDate ###
Information only: Date on which rights terminate.

### Contract.ExpirationMonths ###
Information only: Sell off period (months)

### Contract.PromotionType ###
Value  | Meaning
------:| -------
1      | Allowed, account (pay)
2      | Allowed, but no need to account (pay)
3      | Allowed and account (pay) as normal (high-priced subdivision)

Sales Channels can be defined as being in this category. See [SalesChan.PromotionType]().

This field provides a method to deal with certain kinds of Sales Channels without having to specify Royalty Rates/Subdivisions (Type=3) for them, or without having to account (show) them at all on statements (Type=2).

### Contract.DeletionType ###
Value  | Meaning
------:| -------
1      | Allowed, account (pay)
2      | Allowed, but no need to account (pay)
3      | Allowed and account (pay) as normal (high-priced subdivision)

Sales Channels can be defined as being in this category. See [SalesChan.DeletionType]().

The use of this flag provides a method to deal with certain kinds of Sales Channels without having to specify Royalty Rates/Subdivisions (Type=3) for them, or without having to account (show) them at all on statements (Type=2).

### Contract.SellOffType ###
Also called Free Goods flag.

Value  | Meaning
------:| -------
1      | Allowed, account (pay)
2      | Allowed, but no need to account (pay)
3      | Allowed and account (pay) as normal (high-priced subdivision)

Sales Channels can be defined as being in this category. See [SalesChan.FreebieType]().

The use of this flag provides a method to deal with certain kinds of Sales Channels without having to specify Royalty Rates/Subdivisions (Type=3) for them, or without having to account (show) them at all on statements (Type=2).

### Contract.ProcessingFlag ###
Value  | Meaning
------:|:---------------------------------------------
1      | To Process
2      | Not to Process

### Contract.PayInterval ###
The payment interval (although it defines for the user the frequency of accounting (paying)) defines for the system when the calculated data can be cleaned off the system, and can be over-ruled by a subcontract value.

English Value | Meaning              | French Value
-------------:|:-------------------- |:------------
M             | Month                | M
Q             | Quarter              | T
H             | Half-Year            | S
I             | Off-Center Half Year | I
Y             | Year                 | A

If the field is not filled here, it must be filled at subcontract level.

### Contract.PayDelay ###
Number of days to delay payment.

### Contract.NegativeSalesFlag ###
The Negative Sales Flag defines what should happen in the event that total sales for a certain article/accounting period are negative and when there are no reserves available to offset the negative total.

English Value  | Meaning                          | French Value
--------------:|:-------------------------------- |:------------
A              | Account - Pay a negative royalty | C
C              | Carry forward to next period     | R

### Contract.RoyaltyPayableFlag ###
Valid values are Y / N.

### Contract.ExecutionDate ###
Information only: Date of signature of contract.

### Contract.ReserveFlag ###
Valid values are Y / N. This means _yes_ or _no_ to the [Contract.ReserveSalesType]().

This Flag overrules any Reserve Code from the Article table with the exception that if the Article has a Priority Code = P. It also can be overruled by the Reserve Flag value at the Subcontract Level (if any).

### Contract.ReserveSalesType ###
This defines the one to which the Y/N of [Contract.SalesReserveFlag]() refers, and can be overruled by a subcontract value.

Value  | Meaning
------:| -------
blank  | all
1      | Home (National) sales
2      | Export sales
3      | International sales

All sales batches carry a Sales Transaction Code, and all transaction codes are defined as being one of these 3 categories. It is possible to set up any combination using the 2 fields: this field and [Contract.SalesReserveFlag]().

### Contract.BagatelleTerrFlag ###
Value     | Meaning
---------:|:---------------------------------------------
A         | Sales of all territories accumulated together
C (or I?) | Each territory accumulated separately

Bagatelles is a separate, optional batch processing step, allowing for a reduction in the number of lines to be accounted (paid) out, when these lines contain very small quantities or royalty value.

The same bagatelle fields exist at subcontract level, and the values there overrule those here. Accumulation for a bagatelle condition can be either A=All territories together or each territory I=Indivdually. The condition itself can check for a minimum limit of either sales Q=Quantities or calculated royalty V=Value.

### Contract.BagatelleValueQtyFlag ###
Value  | Meaning
------:|:---------------------------------------------
Q      | Quantities
V      | Values

See [Contract.BagatelleTerrFlag]() for an explanation of how this flag is used.

### Contract.BagatelleLimit ###
Either a sales quantity or royalty value, depending on [Contract.BagatelleValueQtyFlag]().

### Contract.TerrRestricted ###
Value  | Meaning
------:|:---------------------------------------------
R      | Restricted 
A      | Allowed
B      | Blocked
U      | Unblocked

The type of restriction applies to the territories found in tables [Contract_Array_Territory]() and [Contract_Array_TerritoryCombo]().

## Contract_Array_TerritoryCombo ##
By a combination of Reporting and Sales Territories (geographical groups (see [Territory]() table) & default territory 000 are allowed), the values from [Contract.TerrRestricted]() can be applied to all sales lines for a particular contract (the same fields exist and override at the subcontract level):

* Block Sales: no processing
* Restrict Sales: change the territory to rest of the world

You can list either territories to be blocked or those to be unblocked, in order to input a shorter list; the same applies to restricted and allowed territories.

### Contract_Array_TerritoryCombo.SalesTerritory ###
See [Contract_Array_TerritoryCombo]() for a description of how this field is used in conjunction with [Contract_Array_TerritoryCombo.ReportingTerritory]().

### Contract_Array_TerritoryCombo.ReportingTerritory ###
See [Contract_Array_TerritoryCombo]() for a description of how this field is used in conjunction with [Contract_Array_TerritoryCombo.SalesTerritory]().

## Subcontract ##
All processing takes place by Contract and Subcontract; this is the key to the [Royalty]() table, in which all processed data is stored, and from which local Royalty Statements may be printed.

### Subcontract.PayInterval ###
The payment interval is mandatory if not in filled at contract level. Any value here overrides the value found in [Contract.PayInterval](), which is where you find the documentation on this column.

### Subcontract.PayDelay ###
Any value here overrides the value found in [Contract.PayDelay](), which is where you find the documentation on this column.

### Subcontract.ProcessingFlag ###
Any value here overrides the value found in [Contract.ProcessingFlag](), which is where you find the documentation on this column.

### Subcontract.RoyaltyPayableFlag ###
Any value here overrides the value found in [Contract.RoyaltyPayableFlag](), which is where you find the documentation on this column.

### Subcontract.NegativeSalesFlag ###
Any value here overrides the value found in [Contract.NegativeSalesFlag](), which is where you find the documentation on this column.

### Subontract.ReserveSalesFlag ###
Any value here overrides the value found in [Contract.ReserveSalesFlag](), which is where you find the documentation on this column.

### Subcontract.ReserveSalesType ###
Any value here overrides the value found in [Contract.ReserveSalesType](), which is where you find the documentation on this column.

### Subcontract.EscalationFlag ###
Valid values are Y / N.

If the contract contains escalation conditions and this flag is set to Y, then the rate information in the subdivision/territory combination will automatically display the additional 4 rate fields for escalating rates.

Also, if this indicator is set to Y, then the [Subcontract.ScalingFactor]() field must be set to either S=Standard or E=Exception:

* Standard values in the configuration table control the value of one sale of that configuration in the accumulation of sales for escalation.
* Exception values override the table values, are contract-specific and are stored here in the subcontract table.

### Subcontract.PilotSubcontract ###
This is not used within the system (optional informational field), but the input here must be either an existing subcontract of the same contract or 9999 (used only in France).

### Subcontract.ReserveFlag ###
If the reserve indicator is filled Y/N, then this means yes or no to the [Subcontract.ReserveSalesType]() in this table.

See [Contract.ReserveFlag]() and [Contract.ReserveSalesType]() for more details.

### Subcontract.FirstRelease ###
If this is yes, and an article linked to this subcontract is also yes, then only a high price (full rate) royalty rate subdivision will be selected in batch processing.

### Subcontract.FixedRateCurrency ###
If the contract calls for payment on fixed rates, then these can be in any currency, which must be specified here (otherwise no currency conversion will take place). A value must exist in the [Territory]() table and the appropriate rate of exchange must exist in the [ExchangeRate]() table.

### Subcontract.ReserveSalesType ###
This field must be filled if [Subcontract.ReserveFlag]() is populated.

See [Contract.ReserveFlag]() and [Contract.ReserveSalesType]() for more details.

## Subcontract_Array_ReserveCode ##
If the sales reserve code at subcontract level is Y then you can fill here specific reserve codes per configuration (useful for different reserve patterns singles vs. albums etc), sales territory (useful for variations in reserve patterns depending on territory), and sales channel (useful for TV vs. normal sales etc.). All the values are checked against the relevant tables. Defaults are allowed.

Any reserve code here overrides the reserve code obtained from the article file record, unless that code has P=Priority on the article.

## Subcontract_Array_ScalingFactor ##
This table contains the Exception Scaling Information if [Subcontract.ScalingFactorFlag]() is set to E. This value is a percentage: when sales are accumulated for escalation, you can opt for 1 sale = 1 escalation unit or a variation (less or more than 1 unit, or even zero %), according to the configuration and sales channel (e.g. singles may not qualify the same as albums, club sales may not qualify the same as normal sales to dealers). This data is specific to this subcontract.

### Subcontract.TerrRestricted ###
This restrict/block flag is also at contract level; this subcontract value overrules if filled.

See [Contract.TerrRestricted]() for details.

The type of restriction applies to the territories found in tables [Subcontract_Array_Territory]() and [Subcontract_Array_TerritoryCombo]().

### Subcontract.MinimumPricePct ###
_Used only in France_.

A _minimum price percentage_ can also be fixed, one per subcontract: for an international sale, if the royalty price (this is the price after all adjustments, packaging etc.), when converted into final (statement) currency is less than the specified percentage of the lowest-priced sale of the same article in the same sales channel for a national sale in the same accounting quarter, then the royalty price is adjusted up to this minimum, and the transaction is recalculated.

### Subcontract.CutRateFlag ###
_Used only in France_.

On the user screen this field appears with a label of `Antiquity conditions : Options:`

### Subcontract.CutRateYear ###
_Used only in France_.

On the user screen this field appears with a label of `Antiquity conditions : Number of years:`

### Subcontract.CutRatePct ###
_Used only in France_.

On the user screen this field appears with a label of `Antiquity conditions : Deduct.%:`

## Subcontract_Array_TerritoryCombo ##
This table is used exactly the same was that [Contract_Array_TerritoryCombo](), except at the subcontract level. The subcontract level, if filled, overrules the contract level.

### Subcontract_Array_TerritoryCombo.SalesTerritory ###
See [Subcontract_Array_TerritoryCombo]() for a description of how this field is used in conjunction with [Subcontract_Array_TerritoryCombo.ReportingTerritory]().

### Subcontract_Array_TerritoryCombo.ReportingTerritory ###
See [Subcontract_Array_TerritoryCombo]() for a description of how this field is used in conjunction with [Subcontract_Array_TerritoryCombo.SalesTerritory]().

### Subcontract_Array_SubstituteSubcontract ###
One subcontract may be switched to another for a period of time.

The information in the article-contract list or the recording-contract list remains the same (it contains the key subcontract, not the substitute), but for a period of time specified with begin/end dates here (these dates are compared with the sales line _sales date_ field), all validation and further processing is then steered via the substitute subcontract number. Up to 5 substitutions are possible.

## Subcontract_Array_Minimum ##
_Used only in France_.

### Subcontract_Array_Minimum.UnitFee ###
_Used only in France_.

A _minimum unit fee_ payable in final (statement) currency can be specified for up to 6 sales channels: if the unit fee calculated according to reported prices (when converted into final currency (as reported on statements) for international sales), is less than the value specified here by sales channel (or default of all asterisks), then the value is adjusted up to this minimum and the transaction is recalculated. This comparison is performed on values before adjustment/reduction for the participation of the contract in the article (or the contract in the recording and the recording in the article).

### Subcontract_Array_Minimum.SalesChan ###
_Used only in France_.

Default of \*\* is allowed.

## Subcontract_Array_Partner ##
One subcontract can be split between more than one partners (up to 16) on a percentage basis.

It is mandatory that rows exist in this table for any subcontract going through batch processing. The percentage may be zero (e.g for a duplicate or courtesy statement).

Partner numbers and contract numbers sometimes have the same value, but this doesn't mean there is any requirement that the numbers be the same or similar.

It is likely that Royalty Statements are printed by Partner. The Partner Information is stored in the Royalty File during processing, but is not used further within TRACS-EU (it may be used in an accounting system to which we send data � e.g. SAP).

### Subcontract_Array_Partner.PartnerNo ###
Partner number from the [NameAddress]() table.

### Subcontract_Array_Partner.PartnerPct ###
One subcontract can be split between more than one partners (up to 16) on a percentage basis.

The percentage may be zero (e.g for a duplicate or courtesy statement).

## Subcontract_Array_ComposerTitle ##
This table contains references to the works recorded under a specific subcontract: these are not recordings in the logical sense of the system; recordings are contained in the recording file.

## Subcontract_Array_ComposerTitle.Title ##
Free format text field -- no validation is done to make sure a recording by this title actually exists in the system.

## Subcontract_Array_ComposerTitle.Composer ##
Free format text field -- no validation is done to make sure a composer with this name actually exists in the system.

## SubcontractType4 ##
The ComposerTitle array is only populated in Adabas type 1 records. That array is copied in memory to the DDM definition for type 4 records when displaying on screen. However, those records never seem to be written to disk as type 4 records by the TRACS-EU Natural programs. Nevertheless, there are a few records in this table (at least in the German database) that are type 4. It is not known how or when these records were created, or how they're used, if at all.

## Subcontract_Array_SubsidiaryRate ##
The [Subdivision_Array_Rate]() table contains the most important conditions required to pay artists for their royalty, normally a percentage rate on a price (less various adjustments & deductions).

This table contains an alternative method of paying a royalty for sales of articles (sales of recordings cannot be processed this way), based on income/the sum received (called the Gross Income) on the sales line.

This table is also known as the _MAX 50_ information.

This table contains for up to six different percentage rates for a subcontract. For each percentage rate (i.e. row in this table), at least one child table (named Subcontract_Array_SubsidiaryRate_Array_childname) must have rows for the particular rate.

### Subcontract_Array_SubsidiaryRate.CompareFlag ###
If this flag is blank, the [Subcontract_Array_SubsidiaryRate.Percentage]() is used in the calculation, instead of the subdivision rate information of the subcontract. This percentage overrules the normal rate calculation.

If this flag is filled, two calculations are nade: the subdivision one and the percentage of receipts one; the results and compared and the greater or the lesser is used, according to the value in this flag.

The calculation using [Subcontract_Array_SubsidiaryRate.Percentage]() is based on a sum received, rather than on prices and sales (which are irrelevant to this method of calculation): the percentage here is applied to the value of the sales line gross income field, reduced still (as in a normal calculation) by the participation % of the contract in the article (since this method still applies to sales of articles).

It is � of course � possible to make a calculation based on a sum received via the subdivision method; so you really need to use this table only if you want to make a comparison between two different methods and pay the lesser or greater result or � occasionally � to input a set of rates payable on income for certain specific territories (overriding the subdivision rates).

### Subcontract_Array_SubsidiaryRate.CompareFlag ###
Value  | Meaning
------:|:-------
blank  | ?
\>     | Greater than
\<     | Less than

### Subcontract_Array_SubsidiaryRate.SublicenseFlag ###
Value  | Meaning
------:|:-------
G      | Group
N      | Non-group
S      | Sublicensee
\*     | default

### Subcontract_Array_SubsidiaryRate.Percentage ###
See [Subcontract_Array_SubsidiaryRate.CompareFlag]() for details on how thie field is used.

## Subcontract_Array_SubsidiaryRate_Array_ProductType ##
The product type is input on the [Article]() table, but as an optional field; there is no further processing behind it, so you must put only an \* in this field.

## Subcontract_Array_SubsidiaryRate_Array_ReportingTerritory ##
For regular sales of articles, if the reporting territory of a royalty line matches with a value in this row (system reacts on first found match), and the corresponding [Subcontract_Array_SubsidiaryRate_Array_SalesChanFlag.SalesChanFlag]() = 1, then the percentage is activated for that line in calculation processing.

## Subcontract_Array_SubsidiaryRate_Array_SalesChan ##
For regular sales of articles, if the sales channel of a royalty line matches with a value in this row (system reacts on first found match), and the corresponding [Subcontract_Array_SubsidiaryRate_Array_SalesChanFlag.SalesChanFlag]() = 1, then the percentage is activated for that line in calculation processing.

## Subcontract_Array_SubsidiaryRate_Array_SalesChanFlag ##

### Subcontract_Array_SubsidiaryRate_Array_SalesChanFlag.SalesChanFlag ###
Value  | Meaning
------:|:--------------------
1      | Account (pay)
2      | Do not account (pay)

## Subcontract_Array_BlockingSalesChan ##
This table allows you to block the processing of transactions for sales channels which � for this specific subcontract � you do not want to report or pay.

The sales lines are blocked during contract validation and a report is available from Batch Reporting on transactions processed in this way.

This function can be useful for some digital sales channels where a (minor) participant (represented by a contract/subcontract) does not receive a royalty for sales made via some channels.

You can input specific channels in this table or ranges of channels (from/to) in the [Subcontract_Array_BlockingSalesChanRange]() table; the values input must exist in the [SalesChan]() Table.

## Subcontract_Array_BlockingSalesChanRange ##
See table [Subcontract_Array_BlockingSalesChan]() for details on how this table works. This table allows you to specify an inclusive range of sales channels, rather than having to specify each one individually.

## Subcontract_Array_TerritoryCombo ##
This table is the **_lowest level of Contract condition information_**.
Here you see the **_Royalty Rates applicable per territory to the related subdivision key_**.

## Subdivision ##
This table contains _the subdivisions contained in a subcontract_.

The Subdivision is the **key to the actual royalty rate information required to account (pay) to the artist**, which is held at the lower level of _territory combination_.

## Subdivision_Array_Rate ##

### Subdivision_Array_Rate.GrossNetFlag ###
Value  | Meaning
------:|:-------
G      | Gross
N      | Net

### Subdivision.SublicenseFlag ###
Value  | Meaning
------:|:-------
G      | Group
N      | Non-group
S      | Sublicensee
\*     | default

## SubdivisionGeneration ##
When you need to add extra conditions, rows in this table are used as a template to create new [Subdivision]() table rows en masse. This is normally done when adding the rate information for a new subcontract.

## SalesHeader ##
Sales are the initial data in any processing of Royalties: no sales, no royalties. Most sales data is loaded automatically, however, it may well be necessary to enquire on or modify this data. It is also possible to load sales manually.

It is possible to include in the Sales File:
* Royalty Sales of Articles (processing type 1, with article as sales line key)
* Royalty Sales of Recordings (processing type 1, with recording as sales line key)
* Copyright Sales of Articles (processing type 2, with article as sales line key)

Sales data is the same for both Royalty and Copyright sales of Articles, and only different for recordings in that the key on the sales line relates to a different format of product (i.e. a recording). The path of batch processing is however different depending on which of the 3 types of sale is being processed.

The Sales File consists of two levels:
1. A batch header, containing general information about one or more sales lines which are grouped together by this header, i.e. there is information which is common to all those detail lines.
1. A detail record, containing specific information, sales quantities, prices etc. for an article number

A batch has to have certain data/parameters which fit all the lines; it is also a grouping for ease of processing.

The following fields are present on both the Sales header and the Sales detail line; in this case, the value on the Sales Detail Line takes precedence over the header value:
* Sales Period Start
* Sales Period End
* Sales Settlement Period
* Sales Date
* Reporting Territory
* Sales Territory
* Currency Code
* Sales Channel
* Source Tax Percentage
* Exchange Rate Number

## SalesHeader.BatchNo ##
Batch numbering is free-format Numeric-7. Any specific coding scheme of batch numbers is locally-driven (there is also some hard-coded coding schemes in the loading of sales batches coming from the Incoming AIF file).

The batch number will be copied into the Royalty File so that it is always possible to see from which sales batch a calculated transaction was processed.

### SalesHeader.BatchType ###
Value  | Meaning
------:|:-------
A      | Article
R      | Recording

Batch Type indicates whether every sales line within the batch will contain a key of an (A)rticle or a (R)ecording: a mix of both within one sales batch is not allowed.

### SalesHeader.ReportingCo ###
The Reporting Company must be in the [Company]() table; it should be defined as a group, non-group or sub-licensee. For manual sales, using power-add, only the company here can be input (there is another company code - the 'selling company' or 'original reporting company' on the sales line detail record).

### SalesHeader.SalesTransCode ###
The Sales Transaction Code is a value from the [SalesTransCode]() table; any input value must be in the table which also defines:
* 1 = Home
* 2 = Export
* 3 = International

Batch processing is often separate for national (including export) and international.
* Yes/No to reserves per transaction code
* Whether specific sales channels are allowed/blocked

### SalesHeader.SettlementPeriod ###
The Settlement Period is used in conjunction with the reporting company and the exchange rate number (all fields here on the sales batch header) to provide an exchange rate for international sales processing, using the Exchange Rate Table.

### SalesHeader.ReportingTerritory ###
If you add a header and detailed sales lines manually via power add in the user interface, the values of the reporting territory, sales territory, and sales channel from the batch header will be carried forward to the first line of sales detail in power add mode.

The allowed values for the territories and the sales channel can be controlled by the sales transaction code.

### SalesHeader.SalesTerritory ###
If you add a header and detailed sales lines manually via power add in the user interface, the values of the reporting territory, sales territory, and sales channel from the batch header will be carried forward to the first line of sales detail in power add mode.

The allowed values for the territories and the sales channel can be controlled by the sales transaction code.

### SalesHeader.RecodeIncome ###
The recode/income option is used in conjunction with the third party article file. If this option is set to yes, in power add of sales lines you should put in the article number as reported by the licensee, and which must exist in the article file/third party article list linked to the reporting company code which is here on the batch header. The system translates the reported article into the processing article on-line as you infill via power add.

### SalesHeader.SourceTaxPct ###
The source tax percentage will be used during the royalty calculation (most liability is paid net of source tax -see also [NameAddress]() table).

### SalesHeader.NetGrossUpgradePct ###
This is an on-line activity performed during power-add, and is used where a reporting company only reports either gross or net sales (Universal Music standard is that gross & net sales must be present for all lines which are reported internationally). If you infill one, it will create the other according to the percentage populated:
* if upgrade = 10(%): Input 100 gross gives 90 net, or input 100 net gives 111 gross
* if upgrade = 100(%): Input 100 gross gives 100 net, or input 100 net gives 100 gross

### SalesHeader.BatchStatus ###
The batch status is very important: sales validation and all subsequent steps are only performed on sales batches which are closed. You can load sales to the system, but delay processing of these by keeping the batch status open.

### SalesHeader.AifSublicenseFlag ###
The AIF sub-licence indicator is copied to the Royalty File (it can be overridden by the value of a similar field in the sales detail line), and is one of the fields used to select the correct subdivision in Contract Validation step (Note: this is not a confirmed fact!)

### SalesHeader.BatchType ###
Batch Type indicates whether every sales line within the batch will contain a key of an A=Article or a R=Recording: a mix of both within one sales batch is not allowed.

## SalesDetail ##
The data is the same for an Article sales line as for a Recording sales line, except for the key field, and is the same for Royalty and Copyright sales lines (only articles allowed for copyright sales lines).

This table contains many fields which also exist on the [SalesHeader]() table so it is not possible to describe which fields are mandatory here.

**_Sales Detail Line value takes precedence over the Sales Header value_**.

The fields that exist on both Sales Header and Sales Detail Line include:
* Sales Period Start
* Sales Period End
* Sales Settlement Period
* Sales Date
* Reporting Territory
* Sales Territory
* Currency Code
* Sales Channel
* Source Tax Percentage
* Exchange Rate Number

### SalesDetail.GrossSalesQty ###
Either this field or [SalesDetail.NetSalesQty]() must be populated. Based on subdivision, the system will determine which one to use.

### SalesDetail.NetSalesQty ###
Either [SalesDetail.GrossSalesQty]() or this field must be populated. Based on subdivision, the system will determine which one to use.

### SalesDetail.IncomeAmount ###
Exists per individual Sales Line but is only populated for information purposes. It is not used within Royalty Processing.

### SalesDetail.GrossIncome ###
This field is used in order to process exploitation which is calculated on a percentage of receipts.

### SalesDetail.AifSublicenseFlag ###
Field used to select the correct subdivision in the Royalty Processing Step - Contract Validation. (Note: This is not a confirmed fact!)

### SalesDetail.AifPriceLevel ###
Value  | Meaning
------:|:-------
H      | Top Price
M      | Mid Price
B      | Budget

### SalesDetail.BlackBox ###
Locally controlled (i.e. controlled by each site). Miscellaneous Data to be carried over from the sales file to the Royalty File.

### SalesDetail.SopOrigin ###
Field used to detect the origin of a SOP (Sales Order Processing) loaded sales line.  SOP sales are physical product sales?
Value  | Meaning
------:|:-------
1      | International
2      | Local owned rep.
       | Not Used

### SalesDetail.SellingCo ###
This field is also known as the Original Reporting Company Number. It is populated when:
* the Sales line is loaded automatically or
* Added/modified using a screen, it cannot be created during sales power-add.
* System searches for the correct subdivision (see subcontract/subdivision) using the company indicator of the original reporting company if populated else the Reporting Company from the Sales Batch Header.

### SalesDetail.ForeignArticleNo ###
This field is also known as the Original Article Number or the Reported Article Number -- it is populated with the third party article number.

### SalesDetail.TopPricePct ###
If a value is not entered in this field during sales load, it is generated using the top price table during sales validation.

Normally for international sales (including Incoming AIF sales) it will be created/stored here, derived from the Incoming AIF file data. But if there is no value here, then the Top Price table is used to calculate top prices (this is difficult for prices not in the local currency).

### SalesDetail.GrossSalesQty ###
Either this field or [SalesDetail.NetSalesQty]() must be populated.

The system will decide according to the subdivision information which of the two is to be used, and store the sales quantity in a new field: [Royalty.ProcessingQty]() for further steps in the system. A blank value in one of the two fields is allowed, and it means zero sales.

### SalesDetail.NetSalesQty ###
Either this field or [SalesDetail.GrossSalesQty]() must be populated.

The system will decide according to the subdivision information which of the two is to be used, and store the sales quantity in a new field: [Royalty.ProcessingQty]() for further steps in the system. A blank value in one of the two fields is allowed, and it means zero sales.

### SalesDetail.GrossIncome ###
This field is entirely different than the [SalesDetail.IncomeAmount]() field.

This is used in order to process exploitation which is calculated on a percentage of receipts. The gross income must represent not necessarily the income received, but the income which would be received if the processing company was paid for 100% of the article: so if the processing company only owned 20% of the article, the income field might contain $50, but the gross income should contain $250 (50 divided by 20%). The gross income should also be gross of source tax (and - according to locally-defined rules, probably also 'service charges').

### SalesDetail.SalesRecordNo ###
This field is created by TRACS-EU and used internally at various points; it is rarely important to know this number, although it is possible to enquire on a specific SRN (Sales Record Number) via the alternative selection PF10 from the main sales selection screen E91M1900.

### SalesDetail.BlackBox ###
This is a field which is locally controlled; it can be brought into the system via the sales file, and is also stored in the royalty file, from where it can be reported out of the system. However, the system does not use it at all.

### SalesDetail_Array_SalesPrice ###
A _price_ really consists of a _Price Basis Code_ and associated price.

At least one of the following categories of price (defined in the [PriceBasis]() table) must be present on the sales line: dealer, retail or invoice. This is validated by the TRACS-EU system for Article Sales but not for Recording Sales.

## Recording ##
The Recording File contains information which can be used in the system:
* for Royalty Processing of Article Sales via the Article Track List
* for Royalty Processing of Sales of Recordings directly
* for Copyright Processing per track/song/publisher within article

The [Article.ArtistRoyaltyStatus]() field defines how that article is to be processed in batch. This can be either:
* Via the Contracts attached to it OR
* Via the Recordings attached to it in the Track Listing. From there the Contracts attached to that Recording in the Recording File.

Refer to the [Article.ArtistRoyaltyStatus]() field for further details on the values associated with that field. This structure of data using the Recording File for royalties is often referred to as _Track-Based
Processing_.

In addition, direct exploitation of Recordings (i.e. broadcasting/synchronisation) can also be processed via the Recording File. In this case the Sales File contains Recording Numbers instead of Article Numbers, and the system then searches the [Recording]() table directly for the contract participation information.

And for copyright, the article track list can also be used to access the recordings, within which song information can be set up (and within song publisher links can be set up).

### Recording.RecordingCodeGroup ###
This identifies the kind of number stored in the [Recording.RecordingNo]() field.

Value  | Meaning
------:|:-------
5      | ISRC number
N      | Non-standard number

### Recording.RecordingNo ###
The **ISRC (International Standard Recording Code)** is a world-wide standard to which Universal Music is committed. If the number stored here is an ISRC number the [Recording.RecordingCodeGroup]() will be 5.

The ISRC number identifies a unique recording and is now used globally. The code is embedded in a digital carrier. The code is 12-character alphanumeric and is normally allocated by the product department of the owner.

If the ISRC code of a recording is not known, but we want to use the recording in processing, we allocate a non-standard number, and the [Recording.RecordingCodeGroup]() will be N.

### Recording.IsrcNo ###
Only populated if [Recording.RecordingCodeGroup]() = N, which means that [Recording.RecordingNo]() does not contain the ISRC number because it was not known at the time this recording was entered into the TRACS-EU system. When the ISRC does become known, it is entered into this field.

The users can request (periodically!) a small program to switch the two numbers so that the ISRC becomes the key to the Recording File, and the other occurrences of the original N (non-standard) recording number (e.g. in track list of an article) are also switched.

### Recording.RecordingPeriodFlag ###
Information only: Not used within TRACS-EU.

It is intended to link the recording period here to contractual conditions in the revised contract file to be implemented in a later version of the system.

### Recording.Duration ###
Duration of the recording in minutes and seconds.

This value is copied over to the Article file track listing field [Article_Link_Recording.Duration](), so it should be accurate as far as possible.

### Recording.ArtistRoyaltyStatus ###
This field works in a similar way to [Article.ArtistRoyaltyStatus](). See that field for a list of valid values.

However here in the Recording File, this field is only active when the sales source is a recording (i.e. the sales file contains a recording number); often no royalties are due to be paid on sales of a recording, where the company owns the copyright and does not pay the artist directly for secondary use etc.

### Recording.MainContributor ###
This field is no longer in use, it is for information only.

### Recording.ComposerName ###
Not in use, for information only.

### Recording.CopyrightDate ###
Not in use, for information only. When a value is supplied, it is the year only -- the month and day always seems to be January 1.

### Recording.ReleaseDate ###
May not be older than the [Recording.RecordingDate](). When a value is supplied, it is the year and month only -- the day always seems to the first of the month.

### Recording.RecordingDate ###
May not be newer than the [Recording.ReleaseDate](). When a value is supplied, it is the year and month only -- the day always seems to the first of the month.

### Recording.ProjectReferenceNo ###
This will be reported on the AIF file to the Center, if a sale of the relevant recording is reported via TRACS-EU.

### Recording.Genre ###
This is the Repertoire Genre, but it is no longer in use.

## Recording_Link_Song ##
This table contains the recording numbers to which a song has been linked.

## Recording_Link_Song.Song_No ##
The main song of the recording (a medley may have several songs in one recording).

Used only for copyright processing.

## Recording_Array_Contributor ##
The information regarding the exploitation of a recording can be reported by contributors linked to that recording. These contributors may be in reality the artists who performed however logically they are different codes, held in the Contributor File - q.v. (\*).

## Recording_Link_Song ##
This table contains a list of Songs linked to the Recording (a medley would have more than one song, so a recording is not always equal to one song).

This table is used for copyright processing only.

It is possible to define in the [Article]() table that the _long route_ will be used for copyright processing of articles: sales of article > article defines long route for copyright >> article track list to recording(s) >> each recording to song(s) >> each song to publisher(s). See the [Song]() table regarding this functionality.

## Subcontract_Link_Recording ##
This table contains the contracts/subcontracts which are payable for a recording if:

* The Track-Based method of processing Article Sales is used in processing sales of the article
* Direct Exploitation of recordings is processed

### Subcontract_Link_Recording.ContractualCo ###
Is set to the same as the [Subcontract_Link_Recording.Company]() (which is the processing company number) unless manually input.

## Song ##
Song information is only used for copyright processing, and even then it is rarely used, since all the TRACS-EU sites which process copyright sales participate in the International (Copyright) Central Licensing Agreement (ICLA), and for reporting under that agreement it is sufficient to use the Article Short Route (see below).

The structure using the Article Long Route was designed for publishers who are not BIEM members and therefore might not be accounted (paid) under the ICLA.

**Note**: Some of the functionality below may not work efficiently. You are advised to contact the TRACS-EU Help Desk if you would like to make use of this functionality.

Sales of articles for copyright are stored in the [CopyrightStatement]() table under a key of publisher number, but the specification of the publisher can be arrived at by one of 2 methods defined in the Article record:

1. Article Short Route:
   * Up to 5 publishers can be attached directly to an article.
1. Article Long Route: using the track list of an Article:
   * attach Recording to Article (a recording attached to an article is called a _track_)
   * attach Song(s) to Recording
   * attach Publisher(s) to Song

(This is not unlike the way Royalties can be processed via directly-linked Article Participation or via the track list and recording participation).

The _main song_ of a recording is stored in field [Recording_Link_Song.Song_No]().

The song file is a copyright-only file, defining the song and the publisher(s) paid for the use of that song. Songs are created under processing company zero, and not under a specific processing company. This is so that song data can be re-used across companies.

### Song.PublicDomainFlag ###
Y/N - default value is N.

If N � Publisher record needs to linked to Song File.

## Song_Link_NameAddressOfPublisher ##
This table contains the publisher(s) payable for this song, the share due for the publisher(s) and a publisher�s reference (for info only within TRACS-EU, may be reported out to the publisher on a direct statement).

If Song is not public domain ([Song.PublicDomainFlag]() = N), then publisher information must be attached (a warning message is displayed in the online maintenance screen).

## Royalty ##
This table contains the results of Royalty Processing.

* The output of the batch production programs are stored here, from the Article Validation Step.
* The Royalty rows are created during this step, when the sales line information is matched with article file information.
* For every occurrence of a contract/subcontract attached to the specified Article (see alternative in paragraph below); a row is created in this table under the key of Contract and Subcontract number. This is sometimes called the _explosion_ of sales lines into contract lines.
  * Alternative: Sales line to Article to Recording to Contract/Subcontract, if processing is steered via the Article Track List, or Recording Number in the Sales table, to Recording table to Contract/Subcontract table if exploitation of recordings directly is processed.
* A new row in this table is then extended with contract and other information during the contract validation, special clause, reserve, and escalation steps, so that a mathematical calculation can be made, the results of which are also stored here.

This table is then used by local procedures for creating Royalty Statements and interfaces, e.g. a financial system such as SAP Accounts Payable.

Royalty rows are present in this table at any time after the Article Validation has been processed, the extent to which the data is filled here will depend on how far you are in monthly/quarterly processing.

The main usage of this table will probably be after you have processed through to the calculation step, because at that point all values will be populated.

The original sales line which creates row(s) in this table may contain an Article or Recording.
Furthermore, an Article may be processed via its own linked participation (contract/sub-contract/share information) or via its constituent recordings (and therefore via the participation linked to those recordings). The information in this table will reflect that source information and the processing route taken in batch processing.

The basics of computing a royalty amount is as follows:

1. [Royalty.ProcessingQty]() is set to [Royalty.GrossSalesQty]() x [Royalty.SalesPct](), OR [Royalty.NetSalesQty]() x [Royalty.SalesPct]()
1. [Royalty.RoyaltyPrice]() = value of subdivision price basis code x equivalent price percentage (normally 100%) x price adjustment (subdivision value) less packaging deduction.
1. [Royalty.UnitFee]() = [Royalty.RoyaltyPrice]() x [Royalty.RoyaltyRate]() x [Article_Link_Subcontract.ParticipationPct]() x [Subcontract_Link_Recording.ParticipationPct]() (where applicable)
1. [Royalty.RoyaltyFee]() = [Royalty.UnitFee]() x [Royalty.ProcessingQty]()
1. [Royalty.SourceTaxAmount]() = [Royalty.RoyaltyFee]() x [Royalty.SourceTaxPct]() (where applicable)
1. [Royalty.RoyaltyAmt]() = [Royalty.RoyaltyFee]() - [Royalty.SourceTaxAmount](). This is what gets paid out.

**Note**: The rules are different for payment on a percentage of receipts, and for options using contractual conditions for minimum royalty and minimum unit fee.

### Royalty.RecordType ###
Record types are split between National (types 1-4) and International (types 5-8), based on the original sales type of the sales lines:
* **Type 1 or 5 - Normally Processed Records**: These records have no errors in validation, and are available to be used through all processing steps. (Validation, Reserves, Escalation, Bagatelles, Calculation, AIF, File Cleaning). They are deleted during file cleaning, if their pay period (quarterly, half-yearly etc.) is due.
* **Type 2 or 6 - Negative Sales Carried Forward**: If your contracts do not permit the reporting of negative sales, Royalty table rows are carried forward to a subsequent accounting period to the extent that the current quarter processing of the article is negative.
* **Type 3 or 7 - Mixed Period Records**: The system is designed to run quarterly, however if you have half-yearly (or even yearly) contracts, and you are processing calendar quarter 1 or 3 (1, 2, or 3 in the case of yearly contracts), then valid records which would otherwise be stored as type 1 or 5 for half-yearly (or yearly) contracts may be stored as 3 or 7 by invoking the _Mixed Period_ option in the System Options Table.
  * During processing of calendar quarters 1 or 3, these records are not updated for reserve, escalation and bagatelle processing (they are only processed for these steps in calendar quarters 2 & 4), however they are calculated so that you can see the financial results of the processing quarter. They are carried forward in quarters 1 & 3 to the following accounting period, so that when you run the first validation of calendar quarter 2 or 4 (only quarter 4 for yearly contracts), these mixed period rows are transferred to royalty type 1 or 5 (or occasionally 4 or 8 if the current data at the time of that validation makes them fall into error), and are then processed normally though all the steps.
* **Type 4 or 8 - Errors & Warnings**: These are detected during the validation steps and are not further processed; they are carried forward to the next validation (which can be in the same or a subsequent processing period).
  * Errors are found where the data required for all processing steps is incomplete; warnings are often because data does not have to be processed (for example a sales channel like promotional copies does not have to be accounted). Warning transactions are deleted during file cleaning.

### Royalty.SalesType ###
Value  | Meaning
------:|:-------
1      | National
2      | Export
3      | International

This value is copied from the [SalesTransCode.SourceType]() field during batch processing.

### Royalty.RateNo ###
A system-only number irrelevant to users. Populated as part of Royalty Batch Processing. Is a key to the [Rate]() table.

### Royalty.SourcePriceBasis ###
If the subdivision price basis is a calculated one (as defined in the [PriceBasis]() table, then this is the original price from which that one is calculated.

### Royalty.OrigSalesChan ###
Can be changed in batch processing by special clauses routine from sales line value to 000 = Rest of the World.

### Royalty.OrigSubContract ###
The Subcontract code which was held on the Article (or Recording) file record; if this original is substituted to another, the new one forms the key in the Royalty table, and the original one is stored for info only here.

### Royalty.BagatelleCalcBasis ###
Denotes on what basis bagatelles are calculated.

Value  | Meaning
------:|:-------
Q      | Sales Quantity
V      | Royalty Fee

### Royalty.RecordSeq ###
Value  | Meaning
------:|:-------
0      | Initial value when royalty file record is created
1      | A record created via intercompany processing
4      | Any record except intercompany after E90B0204 step

### Royalty.ProcessStepNo ###
Value  | Meaning
------:|:-------
A      | Article Validation
B      | Intercompany Processing
C      | Contract Validation
D      | Negative Sales Processing
E      | Reserves Creation
F      | Reserves Release
G      | Bagatelles
H      | Escalation
I      | Royalties Calculation

If you look at the Royalty table data after calculation, all processed lines will contain step I; but errors and warnings will contain step C (possibly B), because they were rejected during validation.

### Royalty.ProcessingMonth ###
Accounting month of this record. For example, if the royalty record is created from a sales record when validating sales that will appear on year 2019 month 7 financial statements, then this field will be set to 201907, regardless of what the actual sales date was. The actual sales date could be months in the past. A sales record which is a correction to a previous sales record will almost certainly have a sales date that is months in the past.

Processing months always seem to correspond to calendar months. For example, processing month 7 will always be July. However, the [Royalty.ProcessingQtr]() does not always correspond to the calendar quarter. The [Site_Array_QtrToMonthMapping]() table determines what the processing quarter is for any given processing month. For Decca, quarter 1 financial statements show sales that were assigned processing months 2, 3, and 4.

The Validation batch process reads sales records and creates new royalty records from those sales records (along with info gleaned from the contract, article, recording, and other tables). A job parameter is passed to the Validation batch process which states what processing month is to be placed on the new royalty records. It used to be that this job parameter was manually entered by the users when submitting the validation batch job. In 2019 the monthly payments project changed how this works so that now the correct processing month is automatically set when the user runs the last batch jobs of the previous month.

### Royalty.Condition ###
Royalty Rate Price Level

Value  | Meaning
------:|:-------
B      | Budget
C      | Compilations
E      | ?
H      | Top
M      | Mid
N      | ?

### Royalty.ProcessingQtr ###
Accounting quarter of this record. For example, if the royalty record is created from a sales record when validating sales that will appear on year 2019 quarter 3 financial statements, then this field will be set to 20193, regardless of what the actual sales date was. The actual sales date could be last quarter, or even earlier. A sales record which is a correction to a previous sales record will most likely have a sales date that is at least one quarter in the past.

[Royalty.ProcessingMonth]()s always seem to correspond to calendar months. For example, processing month 7 will always be July. However, this field does not always correspond to the calendar quarter. The [Site_Array_QtrToMonthMapping]() table determines what the processing quarter is for any given processing month. For Decca, quarter 1 financial statements show sales that were assigned processing months 2, 3, and 4.

The Validation batch process reads sales records and creates new royalty records from those sales records (along with info gleaned from the contract, article, recording, and other tables). A job parameter is passed to the Validation batch process which states what processing quarter is to be placed on the new royalty records. It used to be that this job parameter was manually entered by the users when submitting the validation batch job. In 2019 the monthly payments project changed how this works so that now the correct processing quarter is automatically set when the user runs the last batch jobs of the previous month.

### Royalty.ProcessingQty ###
Royalties are paid on either the Gross or Net Sales Quantity; however, the Subdivision Sales Percentage can modify the selected quantity. The net result of this calculation is stored here.

This field is calculated either:
* before Escalation Processing (so that escalation is performed on sales quantities net of the sales percentage factor)
* before Royalties Calculation processing (so that escalation is performed on the original sales quantity before the sales percentage factor)

These two options are steered by the value in field [CompanyOptions.SalesPctBeforeEscalation]().

In the event that a Reserve is deducted from this record, this field will be the result of either gross or net sales (according to the contractual G/N indicator) minus the Reserved Quantity, subject (provided escalation and/or calculation has been processed) to the sales percentage.

In the event that this row is a released reserve, Processing Quantity = Released Quantity x Sales Percentage Factor.

If this row is a Released Reserve, the [Royalty.ToBeReleasedMonth]() and [Royalty.ReleaseReasonCode]() will be populated.

Usually, this field is computed to be either:
* [Royalty.GrossSalesQty]() x [Royalty.SalesPct](), or
* [Royalty.NetSalesQty]() x [Royalty.SalesPct]()

## Royalty.PayDelay ##
This field was added as part of the Monthly Payments project.

## Royalty.RecalcDate ##
This field was added as part of the Monthly Payments project.

## Royalty.RecalcUser ##
This field was added as part of the Monthly Payments project.

## Royalty.SentToGL ##
This field was added as part of the Monthly Payments project.

## Royalty.BagatelleFlag ##
Royalty rows with a value of Y are excluded from the Earnings to GL calculations.

## Reserve ##
When Royalty Sales are reserved, the data is stored in this table.

Normally, validation stores data in the Royalty File by contract, subcontract, and article. When sales quantities are reserved, a validated record is moved from the [Royalty]() table to this table; these two tables have similar columns.

A row is stored in this table also by contract-subcontract-article, but additionally by [Reserve.ToBeReleasedMonth](): whenever a reserve is made, the due date at which that reserve will be normally liquidated (reserves may be liquidated before the due date because of negative sales) is stamped onto the record, and forms part of the search key in the Reserve table.

### Reserve.ReleaseCode ###
This field must be filled in order for a reserve to be released. The values are:

Value  | Meaning
------:|:-------
X      | Released because date is due
N      | Released because of negative sales

### Reserve.ReserveMonth ###
The processing month during which the reserve was initially created.

### Reserve.ToBeReleasedMonth ###
The month (i.e. processing month) calculated by the system at which the reserve will normally be liquidated.

### Reserve.ReserveType ###
Value  | Meaning
------:|:-------
1      | Reserves are deducted quarterly for up to 9 quarters (as specified in table 11) following the release date (release quarter plus 8 further quarters); these reserves are progressively released a certain number of quarters (as specified in table 11) after the Sales Date.
2      | Reserves are deducted quarterly as in type one; these reserves are progressively released a certain number of quarters (as specified in table 11) after the Release Date.
3      | Reserves are deducted quarterly forever (over the life of the contract). At present the reserve percentages are those specified in table 11 for quarters 1 to 9 following the release date, and after quarter 9 the reserve percentage applicable to Q9. Reserves are progressively released a certain number of quarters (as specified in table 11) after the Sales Date. If any reserve % field is zero, system should take the last non-zero one.
4      | Reserves are deducted quarterly as in type one; these are all released together after the expiry date and the expiry period of the contract have been reached.
5      | Reserves are deducted quarterly for ever ("over the life of the contract"). At present the reserve percentages are those specified in table 11 for quarters 1 to 9 following the release date; if any of the percentages is zero, then the first non-zero percentage is taken. All reserves are release together as in type four. If any of the reserve % fields is zero, system should take the last non-zero one and not the first.
6      | Copyright Reserve

## Escalation ##
Escalation allows for a change (usually to a higher rate) based on the quantity of sales achieved.

A set of rules for accumulating sales and applying a different rate are mostly held in one or more
escalation clauses at the subcontract level (the scaling rules on configurations and sales channels are
held at the subcontract level).

The clauses are examined in numeric sequence during the batch processing of escalation, so some
information (e.g. the same article or recording) may exist the same in more than one escalation clause
within the subcontract, but it will be the first clause which matches the sales/contract data in the
transaction being processed which will be used for calculating escalation (no transaction can be used
more than once for escalation).

Escalation is activated by sales of articles or recordings, and the articles/recordings which apply to the selected clause are mentioned in the article list.

An alternative to escalation on specific target quantities which are related to the specific clause or subcontract is that escalated rates can be payable when sales in a specific territory reach an 'award level', normally gold, silver and platinum. These vary by territory and the values are stored in the Territory Table. You cannot enter target quantities if you enter award levels, and vice-versa.

An additional possibility is to pay different (normally greater) sales percentages according to the target quantities achieved.

It is possible to control the royalty rate payable in one further way: to define a target quantity after which the royalty rate reverts to the normal (un-escalated) rate: this is the "To quantity".

### Escalation.ConditionSerialNo ###
The escalation clauses are examined in numeric sequence as defined in this column during the batch processing of escalation, so some information (e.g. the same article or recording) may exist the same in more than one escalation clause within the subcontract, but it will be the first clause which matches the sales/contract data in the transaction being processed which will be used for calculating escalation (no transaction can be used more than once for escalation).

### Escalation.AccumulationFlag ###
Accumulation of sales quantities for qualifying articles can be made in one of three ways:

Value  | Meaning
------:|:-------
A      | (A)ll articles & territories together (Worldwide)
I      | Each article (I)ndividually, all territories together
T      | Each (T)erritory individually, all articles together

### Escalation.AchievedRank ###
You can enter here manually (for information only) the Escalation Level (see: rate index) which has been reached, or simply Y/N. This might be useful if there are already sales accumulated before you create the system data (e.g. on first installing the system). This field is over-written by a value calculated by the system when the escalation processing is run.

Value  | Meaning
------:|:-------
Y      | ?
N      | ?
1      | ?
2      | ?
3      | ?
4      | ?

### Escalation.ApplicationStartingMonth ###
If a sales month is between this date and [Escalation.ApplicationEndingMonth](), apply all the parameters of the clause as normal.

### Escalation.ApplicationEndingMonth ###
If a sales month is between [Escalation.ApplicationStartingMonth]() and this month, apply all the parameters of the clause as normal.

### Escalation.EffectiveStartingMonth ###
If a sales month is between this date and [Escalation.EffectiveEndingMonth](), apply only the first escalation rate, and ignore _other information_ parameters.

### Escalation.EffectiveEndingMonth ###
If a sales month is between [Escalation.EffectiveStartingMonth]() and this date, apply only the first escalation rate, and ignore _other information_ parameters.

### Escalation.PreviousQty ###
The sales quantity accumulated up to/including any previous processing of escalation; this value is updated with the accumulated sales of the current period when File Cleaning is run.

If you are setting up an escalation clause with accumulation option = ALL, but previous sales have already accumulated outside of the system, you should fill the quantity here, and it will be taken into account in the accumulation in the next batch processing run of escalation. This value is only displayed when the [Escalation.AccumulationFlag]() = A.

### Escalation.AccumulatedSalesQty ###
This is the amount accumulated by the system to date, including any current period escalation processing and the previous sales amount from a prior period. This value is only displayed when the [Escalation.AccumulationFlag]() = A.

## Escalation_Array_Rate ##
When batch processing validates contract data, it stores in the [Royalty]() table all the possible rates (i.e. [Escalation_Array_Rate.RateIndex]() 0-5 where filled); batch processing of escalation results in the selection of one of these indices for calculation of royalties, according to the quantities achieved.

### Escalation_Array_Rate.FromQty ###
The value in this field denotes the first accumulated sales for which a different rate index is payable. If you input here e.g. [Escalation_Array_Rate.RateIndex]() = 1 from target quantity 100,000, then the normal ([Escalation_Array_Rate.RateIndex]() = 0) territory combination rate will be paid for sales from 1 - 99,999; sale 100,000 and upwards will be paid at the first escalation rate. This is possible for up to 4 escalation rates.

### Escalation_Array_Rate.RateIndex ###
The rate index can be from 0 (or blank) to 5 inclusive, and relates to the actual royalty rates as displayed in the territory combination screen, where:

Value  | Meaning
------:|:-------
0      | normal royalty rate
1      | 1st esclation rate
2      | 2nd esclation rate
3      | 3rd esclation rate
4      | 4th esclation rate

## PriceUplift ##
Holds a number of attributes that contribute as a key to define the ratio between Dealer and Retail Prices.

## ManualAdjustment ##
Defines the manual adjustment codes and the associated warning/memo messages that can be attached to subcontracts.

## ArticleStructure ##
Contains defined business rules relating to the format of an Article Number. It can be used to do on-line checks on Article File data as it is input, and to automatically create part of the article file number.

## NameAddress ##
The structure of the contract information is such that - although royalties are calculated by contract and
subcontract � TRACS-EU additionally stores with each contract/subcontract transaction (in the Royalty File)
the subcontract partner information.

A partner is essentially a name, stored here in the name & address file. Although the data is calculated
per contract & subcontract, usually (but not mandatorily) Royalty Statements and any Royalty Payments
are created under the key of the Partner/Name. In most sites, at least one Partner is mandatory per
subcontract (input on the 'P' screen E91M1514 of the subcontract), and is a validation error if there is no
partner link.

The Name & Address File is also used to store Copyright Publishers.

**Note**: Very little of the data in this table is actually used by TRACS-EU, but is mostly used by outgoing interfaces such as Royalty Statements.

### NameAddress.SourceTaxFlag ###
Valid values are Y / N.

Most Royalties are paid net of Source Tax. However, this flag allows the user to define the Source Tax is to be deducted from a Partner.

The tax is always deducted during Calculation (as long as the information is supplied on the original sales line); however, if the [NameAddress]() table says (N)o to a specific partner, then the liability reports are produced with values before tax deduction, and of course Royalty Statements can also be produced without tax deduction where applicable.

If this field is 'Y' then the Earnings Interface to GL calculation of the liability amount is based on the [Royalty.RoyaltyAmt], otherwise it is based on the [Royalty.RoyaltyFee].

### NameAddress.VatFlag ###
Valid values are Y / N.

Not used within TRACS-EU but may be used for local statements.

### NameAddress.StatementType ###
Value must exist in [StatementType]() table.

Codes can be specified for different types of statement layout.

Creation of Royalty statements is not part of the core TRACS-EU system.

### NameAddress.GroupId ###
This field is not currently being used.

### NameAddress.PublisherSocietyFlag ###
Valid values are Y / N.

The Publisher fields in this table relate to Copyright processing only. Statutory Rates and copyright-related Article & Song information check this flag for the status of any name used in those areas of the system.

### NameAddress.PublisherProcessingPeriod ###
Value  | Meaning
------:|:-------
Q      | Quarterly
H      | Half Yearly

### NameAddress.TerritoryCode ###
Territory (location) this Name/Address is in. This is a mandatory field since it is sent to SAP.

## PartnerDetail ##
This table was originally used only by the UK, but now contains some information to be used by other sites who use the SAP application for financial control of Royalty Earnings & Payments.

### PartnerDetail.PartnerType ###
Value  | Meaning
------:|:-------
A      | Artist
C      | Complimentary
I      | International
P      | Producer
D      | Deductible Producer

Type C and I do not create a liability to be posted to SAP.

### PartnerDetail.PayeeName ###
This will be used to make the payment from the accounting/payment system.

### PartnerDetail.Subledger ###
The (old) JDE Profit Centre and Advance Subledger references are input here. They are checked against external (non-TRACS-EU) table data to ensure that they exist elsewhere.

This field will be filled automatically if [PartnerDetail.PartnerType]() = D and the first deductible reference is filled.

### PartnerDetail.VendorCode ###
This is an SAP value.

It may not be known at the time a new Partner is created in TRACS-EU, and may need to be added later when it has been generated by SAP.

### PartnerDetail.WithholdingTaxFlag ###
Valid values are Y / N.

This is the **local** tax (not that applied on non-local transactions and deducted from Incoming AIF receipts etc) deducted from royalty payments to certain countries (local rules apply on this).

### PartnerDetail.NoOfStatementCopies ###
This refers to the SAP Summary Statement, you can define how many copies are produced from SAP here.

### PartnerDetail.Frequency ###
For example H90 = half-yearly accounting within 90 days

### PartnerDetail.AuthorizationLevel ###
An indication of the importance of an account, where 1 is the highest.

### PartnerDetail.PartnerCode ###
If this is D then one account must be populated.

## CopyrightReserve ##
Data is stored in this table as a result of the copyright batch processing steps.

The data held here can be either:
* Record Type 1: a regular reserve (i.e. a quantity of sales withheld against future returns)
* Record Type 2: a negative sale, to be carried forward to a subsequent period, to be (hopefully) offset against future positive sales (see copyright batch processing regarding how this data is used)

It is not possible to simulate batch processing and manually move a record in or out of this table. If you wish to create your own manual reserve file data, you must create a new record in the reserve file and modify or delete the corresponding [CopyrightStatement]() table row to reduce the sales quantity due to be accounted for the reserve quantity created. Similarly if you wish to liquidate a reserve before the due date you will have to create a manual entry in the [CopyrightStatement]() table corresponding to the quantity manually released from reserves. It would be necessary to create new/additional records in either file if you want to create or release a partial reserve.

## CopyrightStatement ##
Copyright is an optional process; some TRACS-EU sites **do not** process copyright sales.

This table contains the results of copyright processing (the results of royalty processing are found in the [Royalty]() table). The output/results of the batch production programs are stored here, beginning with the copyright validation step TR0B9600. Rows are added to this table during this step, when the sales line information is matched with article file information (for the short route process for copyright, which is the normal method); for every occurrence of a publisher attached to the specified article (see alternative in paragraph below), a row is created in this table under the key of publisher number.

* Alternative: sales line to article to recording to song to publisher, if it is required to steer processing via this long route, using the article track list; it is not possible to process copyright sales of a recording via TRACS-EU). This long route is rarely used.

This row is then extended with statutory rate and other information during the validation, reserves/negatives and calculation steps, and the final calculated result is stored here. This table is then used (usually) to send the copyright reporting file to the Center for transmission to the International Copyright Licensing partner.

Data can be seen in this table at any time after the copyright validation has been processed, the extent to which the data is populated here will depend on how far you have processed your quarterly information. The main usage of this file will probably be after you have processed through to the calculation step, because at that point all values will be populated.

## BatchControl ##

### BatchControl.JobStatus ###
Value  | Meaning
------:|:-------
blank  | No action is available for this user on this option.
S      | Submitted: The job has been scheduled for submission but has not started yet.
E      | Executing: The job has been started and is processing now.
F      | Finished: The job has run successfully with no system errors reported.
I      | Interrupted: This status shows that the job has been interrupted to perform some backup or other actions.
A      | Aborted: A system error has occurred.

## CompanyOptions ##
This table provides the user the ability to define system default values and functionality options that are automatically apply to the system and its processes.

### CompanyOptions.NationalityOnline ###
This field sets the language in which the start-up menu is displayed, since at that point the language of you as a specific user is not yet.

Currently English and French are supported. This field will be overruled by a nationality found in [UserInfo.Language]().

### CompanyOptions.ProcessingNationality ###
This nationality is used in the batch production.

Certain processing inside the programs is steered by nationality. See the batch production functional specification for more details.

### CompanyOptions.ArticleRequiredOnSalesLine ###
When a sales line is entered in manual sales maintenance the article is/is not required to be already present in the Article file.

### CompanyOptions.ContractRecordingLink ###
Allows a Recording Number to be linked to a Contract in the Article Contract List.

This facility is only used by Philips Classics to produce their royalty statements.

### CompanyOptions.SalesPctBeforeEscalation ###
This function determines whether deduction of sales percentage should be done before or after escalation or not at all.

### CompanyOptions.CompanyForCurrencyConversion ###
This defines whether the exchange rate is to be selected according to the reporting company on the sales batch header record or (where filled) the selling company on the sales line detail record (the other parameters: settlement period and processing number are also present on both header and detail records).

### CompanyOptions.ReserveAccumulationFlag ###
When Reserves are accumulated and a percentage withheld, the accumulation is made by Subcontract and Article number, and also then separately by either just sales type (home, export, other) or optionally by sales channel within sales type.

### CompanyOptions.ContractualCoReqd ###
This option sets the requirement to enter a Contractual Company in the Contract file, makes the field mandatory.

### CompanyOptions.ContractParticipationPct ###
Indicates whether the contract participation % for a specific contract/subcontract line in the contract list of the article file can be greater than 100%.

### CompanyOptions.TotalContractParticipationGreaterThan100Pct ###
Indicates whether the total of all participation percentages of all contracts in the list can be greater than 100%.

### CompanyOptions.ContractCoRequired ###
Indicates whether the contractual company field in the contract list of an article is mandatory or not.

### CompanyOptions.RoyaltyCalculationFlag ###
This option indicates whether a Royalty Calculation has to be made for international repertoire to be reported to Baarn.

If Y, a royalty row will be included in the Earnings to GL interface calculations provided other selection criteria are met. Otherwise, [Royalty.ExchangeTape] must be zero or blank to include the royalty row in the calculations.

### CompanyOptions.PartialProcessingOfSalesLine ###
When a sales line is expanded in the royalty file, it is possible to have some properly validated contracts and some with errors.

With this option you can decide to process all the properly validated contracts or only all contracts for complete sales lines together.

If you choose yes, then sales lines are deleted at the end of the processing quarter during File Cleaning, but contract errors remain on file without the original sales information being present any more.

### CompanyOptions.MixedPeriodProcessing ###
If set to 'Yes' - Contract which have a pay-period greater than quarterly (so half-yearly or yearly), are not subject to reserve, escalation or bagatelle processing until the final quarter of their payment cycle.

E.g. Half-yearly contracts are not processed through these options in quarter one or three) and during that final quarter the contracts are reprocessed with the contract data valid at that later time.

A Royalty Calculation is however always made on these lines.

### CompanyOptions.DeductionsFromSourcePrice ###
This option is especially for PIM Pop it defines that all deductions are made of the Original Source Price value, rather than being made sequentially one after the other.

### CompanyOptions.VideoSpecialFlag ###
This option is used only by PVI to allow them two separate ways of calculating on income received; set this option to N Debugging start and end (only on company 0000000).

Input here is a sales record number. When one of the two fields is not zero (empty) the debugging tool built in the system takes affect. An extensive report will be printed to printer 9 which should be defined at that point in your JCL. This report will list all program names executed, together with the data records processed.

A start and end internal system sales line number can be supplied. Only sales lines with a number in the specified range will be executed if they also would be processed normally. Sales lines outside the specified range will never be processed under these conditions.

## Rate ##
Statutory rates will be held in this table by publisher under company 0000000.

This table is used within the Validation Step, to check that the publisher(s) attached to either 1) Article File record (short route) or 2) Song File record (long route) contain at least one Statutory Rate that exists within this table.

The publisher must also exist in the [NameAddress]() table under the specific processing company code (i.e. the code pertaining to the sales record).

The batch process will use the various elements from the Sales and Article tables which form the key to a Statutory Rate table record to find the applicable rate, and use that rate within processing.

## ContractType ##

### ContractType.IncomeFlag ###
If true, this is considered to be an income calculation only contract, and royalty rows for this contract will be ignored on the liability reports.

In the Earnings to GL interface, only royalty rows for contracts with this field set to false are included in the calculations.
