## Offline Conversion Integration API

The current agreed workflow of offline conversion download from customer's data source and subsequent upload to FB of looks like this:

```
-----------------        CUSTOM API        --------------      COMMON API     -----------------------      FB API      ------
| CLIENT EVENTS |  --------------------->  | COMMON API |  ---------------->  |       UNICORN       |  ------------->  | FB |
|  DATA SOURCE  |   (REST/XMLRPC/SOAP..)   |  ADAPTER   |    (FTP or SFTP)    | (FB CONV. UPLOADER) |       HTTP       ------
-----------------                          --------------                     -----------------------
```

This document is the specification of the **COMMON API** that is used by Unicorn to read pre-processed data from the adapter.

### API Protocol and File Structure

The adapter uploads data files to **FTP** or **SFTP** server. The FTP/SFTP server is continously examined by Unicorn, if it detects a new file or a modified file, the file is processed. Therefore, each file should only contain events that are to be uploaded to FB, no modified events are allowed (because FB does not permit editing conversion events). So the FTP/SFTP server could look like this:

```
.
├── 2018-01-01.csv
├── 2018-01-02.csv
└── 2018-01-02-append.csv
```

### File Format

Each file uploaded to the FTP/SFTP server must be in **CSV** fromat and the CSV column separator must be the comma character (`,`). The file must also contain a single header with names of the columns and any number of data rows. There are mandatory columns and optional columns. The order of the columns is not important.

#### Mandatory columns

* `event_time` - Time when the conversion event occured, must be in `yyyy-MM-ddTHH:mm` format, so e.g. `2019-02-12T20:01`.
* `value` - Transaction value. Must be a real number (double or single precision), e.g. `20.234`.
* `currency` - Transaction value currency in ISO 4217 alpha code, e.g. `CZK`, `EUR`, `USD`, `AED` etc.

#### Optional columns

 * `event_name` - Usually `Purchase` (used implicitely if `event_name` not present), but can be also `ViewContent`, `Search`, `AddToCart`, `AddToWishlist`, `InitiateCheckout`, `AddPaymentInfo`, `Lead`, `CompleteRegistration` or `Other`.
 * `email` - e.g. `customer@client.com`
 * `phone` - Phone number of the customer with no symbols, characters or leading zeros, e.g. `55081994960230`. If column `country` is not specified, should contain also country code number prefix (e.g. `420`).
 * `gender` - Customer's email, either `m` for male or `f` for female.
 * `date_of_birth` - Date of birth of the customer in `yyyy-MM-dd` format, e.g. `1991-10-05`.
 * `last_name` - Customer's last name, only one name.
 * `first_name` - Customer's first name, can be multiple names e.g. for full name `Rosane Aparecida Antunes`, the first name is `Rosane Aparecida` (and last name is `Antunes`).
 * `city` - City of the customer, e.g. `Prague`.
 * `zip` - Zip code of the customer.
 * `country` - Customer's country code in [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) code, e.g. `CZ`.
 * `state` - For US states, use [2-character ANSI abbreviation code](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standard_state_code). For states outside US, use state name with no punctuation, no special characters, no white space.
 * `madid` - Mobile advertiser id.
 
Usually, most of these columns will be not present or empty (because the client's data source does not have them), but at least either an `email` or `phone` must be present (otherwise the FB won't be able to match such events to real users).
 
Example of a CSV file in this format:
 
```
email,phone,first_name,last_name,country,event_time,event_name,value,currency
customer1@client.cz,774123456,Pavel,Novak,CZ,2019-01-12T20:01,Purchase,1999.5,CZK
customer2@client.com,553484729328,Ahmed Mohamed,Shahrani,AE,2019-01-13T10:01,Purchase,99.99,AED
customer3@client.com,971794000539123,Mohyiddine,Soltani,KW,2019-01-14T11:21,Purchase,300.00,KWD
```
 
#### Sensitive Data Columns

Sometimes, the client does not wish to disclose sensitive customer's data with us and instead provides us with already hashed data. In such a case, append `_hash` to the column name, e.g. `email_hash`, `phone_hash`, `country_hash`. These columns, however, cannot be hashed: `event_time`, `value`, `currency`, `event_name`.

#### Extra Columns

The client might want to use some of their data to employ a more complicated logic, e.g. upload events from certain stores to a different offline FB event set (done by Unicorn). Unicorn, however, needs this data in the csv file to be able to work with it. Therefore, it is possible to add additional columns to the csv output. These column names must not clash with already established column names (mandatory or optional). Examples of allowed extra columns: `source`, `brand`. Disallowed extra column names: `country`, `event_time` etc.
