# Sample-ComplianceSearch-OnPrem

## Permissions

First you might need Mailbox Import Export permissions to be able to run and export results with New-MailboxSearch (last step on the below procedure). Here's a way to give a user or yourself the permissions to export search results, in the below example for the user samdrey of the CanadaDrey domain:

```powershell
# Create a new Role Group with a meaningful name of your choice
New-RoleGroup "Import and Export permissions"
# Add the "Mailbox Import Export" built-in role to this new Role Group
New-ManagementRoleAssignment "Import Export Assignment" -SecurityGroup "Import and Export permissions" -Role "Mailbox Import Export"
# Add a user or yourself as a member of this ROle Group
Add-RoleGroupMember -Identity "Import and Export Permissions" -Member canadadrey\samdrey
```

## Sample

Here is a sample Compliance and eDiscovery search example

```powershell
# Placing the search parameters in variables for convenience (avoiding to writing the same strings several times)
# The name of the search (and the eDiscovery New-MailboxSearch later on)
$SearchName = "e-Mail With Attachment MyFile.pdf"
# The query string in Keyword Query Language
# For mail specific parameters, you can use the operators (AND, OR, ...) as well as the following properties:
# AttachmentNames, Bcc, Category, Cc, Folderid, From, HasAttachment, Importance, IsRead, ItemClass, Kind, Participants, Received, Recipients, Sent, Size, Subject, To
$Query = 'sent>=01/01/2023 AND sent<=06/30/2023 AND attachmentnames:MyFile.pdf'

# Step 1 - Creating the Compliance Search

New-ComplianceSearch -Name $SearchName -ExchangeLocation all -ContentMatchQuery $Query

# Step 1 bis - Starting the compliance search

Start-ComplianceSearch -Identity $SearchName

# Step 2 (Optionnal) - Counting the number of mailboxes with results in these - check you have less than 500 with results, otherwise
# you might need to narrow down the search with a better KQL query, or create groups of mailboxes with Distribution Lists, and use
# -ExchangeLocation <Distribution List Name or SMTP address> with New-ComplianceSearch cmdlet.

.\Count-SourceMailboxes.ps1 $SearchName

# Step 3 - Run the script to search and estimate or copy the results in a Discovery Mailbox (to later export these to a PST file)

# To estimate only the search results

.\RunSearch.ps1 -SearchName $SearchName

# To copy the results into a target mailbox

.\RunSearch.ps1 -SearchName $SearchName -TargetMailbox Discovery.Mailbox01@contoso.ca -CopyResults
```

