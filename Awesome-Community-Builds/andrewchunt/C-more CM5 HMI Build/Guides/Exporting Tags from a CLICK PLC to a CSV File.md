# Exporting Tags from a CLICK PLC to a CSV File

The CLICK Programming Software allows you to export PLC tags and their associated nicknames to a CSV file. This is especially useful when configuring an HMI, such as a C-more HMI, because the exported information can be used to help build your HMI tag database.

Before You Begin — Assign Nicknames

For the easiest export process, make sure that all PLC tags you intend to use have a Nickname assigned in the CLICK project.

A Nickname provides a descriptive name for a PLC address instead of relying only on addresses such as X001, Y001, or C1.

For example:

| PLC Address | Nickname |
|-------------|----------|
| X001 | Stop Button |
| X002 | Start Button |
| X003 | eStop Reset Button |
| Y001 | Fast Run Command |
| Y002 | Stopped Light |
| Y003 | Running Light |
| Y004 | eStop Reset Light |
| C1 | HMI Stop |
| C2 | HMI Start |

Using consistent and descriptive nicknames also makes it much easier to identify and map the tags when they are later imported into the C-more HMI project.

## Export the Nicknames

Open the CLICK Programming Software and load your PLC project.

Verify that the PLC addresses you want to use with the HMI have Nicknames assigned.

From the CLICK Programming Software menu, select:

File → Export → Nickname

Choose the location where you want to save the exported file.

Give the CSV file a descriptive name, such as:

PLC_OT_Lab.csv

Complete the export.

Open the resulting CSV file in a text editor or spreadsheet application to verify that your PLC addresses and nicknames were exported correctly.

## Why Use Export Nickname?

Using Export Nickname is an easy way to get the PLC tags you have already defined in your CLICK project into a CSV file.

Instead of manually recreating tag names for the HMI, the exported nicknames can be used as the starting point for building the C-more HMI tag database.

Important: Assign a Nickname to all of the PLC addresses you plan to use before performing the export. This helps ensure that the tags you need are clearly identified in the exported CSV file.

## Next Step — Importing into C-more

The CLICK Nickname CSV is the starting point. The exported information can then be prepared for import into the C-more Programming Software, allowing the HMI tags to correspond with the same descriptive names used in the PLC program.

This makes the PLC and HMI projects easier to understand, maintain, and modify as you experiment with your OT Lab.