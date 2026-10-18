# Car Crash – 

**Challenge:** Car Crash  
**Category:** OSINT  
**Points:** 50  
**Author:** @RP-01?  
**Event:** DEADFACE CTF

## Challenge Description

DEADFACE has been researching companies connected to one of its latest targets. Among the recovered files was a photograph of a traffic accident involving a commercial truck.

Our task was to identify the company that owns the truck and determine when that company was founded.

**Flag Format:** `deadface{MM_DD_YYYY}`

## Investigation

### Step 1: Analyze the Photograph

The provided photograph showed a traffic accident involving a white commercial truck.

Before investigating the truck, I examined the surrounding environment for geographical clues.

Some identifiable details included:

- S 200th St road sign
- Westwood Square shopping center
- 76 gas station
- Several recognizable businesses, including Safeway and Ross

These clues suggested that the accident occurred in the Seattle–Tacoma metropolitan area of Washington State.

### Step 2: Identify the Truck Using USDOT Number

The most important clue was printed on the side of the truck's cargo box.

The visible USDOT number was:

```text
USDOT: 080806
```

Since USDOT numbers are used to identify registered motor carriers in the United States, this provided a direct way to investigate the company.

I searched the number in the official FMCSA SAFER database.

**Website:** https://safer.fmcsa.dot.gov/

The carrier record identified the company as:

**J.B. Hunt Transport Inc.**

### Step 3: Find the Company's Founding Date

After identifying the company, the next step was to determine its founding date.

I searched for J.B. Hunt's corporate history and checked its official SEC filings.

The company's annual report confirmed that J.B. Hunt was incorporated in Arkansas on:

```text
August 10, 1961
```

This provided the exact date required by the challenge.

## Final Answer

Converting the date into the required `MM_DD_YYYY` format:

```text
08_10_1961
```

**Flag:**

```text
deadface{08_10_1961}
```

## References

- [FMCSA SAFER – Company Snapshot](https://safer.fmcsa.dot.gov/)
- [J.B. Hunt Investor Relations](https://investor.jbhunt.com/)
- [SEC EDGAR – Company Filings](https://www.sec.gov/edgar/search/)

## Conclusion

This challenge demonstrated how small details in an image can become valuable OSINT clues. By extracting the USDOT number from the truck, identifying its registered carrier through FMCSA, and verifying the incorporation date through an SEC filing, I was able to recover the flag.

**Key Takeaway:** Never overlook vehicle identification numbers, business signage, or other seemingly minor details when conducting image-based OSINT investigations.
