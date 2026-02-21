---
name: salesforce-dx
description: "Salesforce DX project structure and metadata architecture knowledge. Use this skill whenever the user mentions Salesforce, SFDX, force-app, metadata structure, LWC components, Apex classes, custom objects, or any Salesforce project work. Also use when navigating or creating files in a Salesforce project to ensure correct placement. Even if the user doesn't say 'Salesforce' explicitly, trigger this skill if you see sfdx-project.json, force-app/, or .cls/.trigger/.page files."
user-invocable: true
argument-hint: "[metadata-type]"
---

# Salesforce DX Project Structure

This skill provides the canonical structure of a Salesforce DX (SFDX) source-format project. Use it to know where every metadata type lives, what files each folder expects, and what naming conventions to follow.

## Root Structure

```
project-root/
├── sfdx-project.json          # Project config (packageDirectories, apiVersion, namespace)
├── .forceignore               # Files excluded from source tracking
├── .gitignore                 # Git exclusions
├── manifest/
│   └── package.xml            # Deployment manifest (lists metadata types + members)
├── config/
│   └── project-scratch-def.json  # Scratch org definition (edition, features)
├── scripts/
│   └── apex/                  # Anonymous Apex scripts (.apex files)
└── force-app/                 # Default package directory
    └── main/
        └── default/           # All metadata lives here
```

## Metadata Folders: force-app/main/default/

### Apex Code

```
classes/                       # FLAT - Apex classes
├── MyClass.cls                # Source code
└── MyClass.cls-meta.xml       # API version + status

triggers/                      # FLAT - Apex triggers
├── MyTrigger.trigger          # Source code
└── MyTrigger.trigger-meta.xml # API version + status
```

### Lightning Web Components (LWC)

```
lwc/                           # Each component = subfolder
└── myComponent/
    ├── myComponent.js             # Controller (required)
    ├── myComponent.html           # Template (required)
    ├── myComponent.js-meta.xml    # Config: apiVersion, targets, visibility (required)
    ├── myComponent.css            # Styles (optional)
    └── __tests__/                 # Jest tests (optional)
        └── myComponent.test.js
```

Naming: camelCase for folder and files. Must start with lowercase letter.

### Aura Components (Lightning legacy)

```
aura/                          # Each component = subfolder
└── MyAuraComponent/
    ├── MyAuraComponent.cmp           # Markup (required)
    ├── MyAuraComponentController.js  # Client controller
    ├── MyAuraComponentHelper.js      # Helper
    ├── MyAuraComponent.css           # Styles
    ├── MyAuraComponent.design        # Design attributes
    ├── MyAuraComponent.svg           # Icon
    ├── MyAuraComponent.auradoc       # Documentation
    └── MyAuraComponent.cmp-meta.xml  # Meta
```

Naming: PascalCase for folder. File stems match folder name.

### Custom Objects

```
objects/                       # Each object = subfolder
└── MyObject__c/
    ├── MyObject__c.object-meta.xml        # Object definition (label, sharing, etc.)
    ├── fields/                            # Custom fields
    │   ├── FieldName__c.field-meta.xml
    │   └── AnotherField__c.field-meta.xml
    ├── listViews/                         # List views
    │   └── All.listView-meta.xml
    ├── compactLayouts/                    # Compact layouts
    │   └── MyLayout.compactLayout-meta.xml
    ├── recordTypes/                       # Record types
    │   └── TypeName.recordType-meta.xml
    ├── validationRules/                   # Validation rules
    │   └── RuleName.validationRule-meta.xml
    ├── webLinks/                          # Custom buttons/links
    │   └── LinkName.webLink-meta.xml
    └── businessProcesses/                 # Business processes
        └── ProcessName.businessProcess-meta.xml
```

Standard objects (Account, Contact, Case, etc.) follow the same subfolder pattern but without __c suffix.

### Visualforce

```
pages/                         # FLAT - Visualforce pages
├── MyPage.page
└── MyPage.page-meta.xml

components/                    # FLAT - Visualforce components
├── MyComp.component
└── MyComp.component-meta.xml
```

### Automation

```
flows/                         # FLAT - Flow definitions (screen flows, auto-launched, etc.)
└── My_Flow.flow-meta.xml

flowDefinitions/               # FLAT - DEPRECATED since API v44. Avoid.
└── My_Flow.flowDefinition-meta.xml

workflows/                     # FLAT - Workflow rules (legacy)
└── ObjectName__c.workflow-meta.xml
```

### UI & Layout

```
applications/                  # FLAT - Lightning/Classic apps
└── MyApp.app-meta.xml

layouts/                       # FLAT - Page layouts
└── ObjectName__c-Layout_Name.layout-meta.xml

tabs/                          # FLAT - Custom tabs
└── MyObject__c.tab-meta.xml

quickActions/                  # FLAT - Quick actions
└── ObjectName__c.ActionName.quickAction-meta.xml
```

### Security & Access

```
profiles/                      # FLAT - User profiles
└── Admin.profile-meta.xml

permissionsets/                # FLAT - Permission sets
└── MyPermSet.permissionset-meta.xml

customPermissions/             # FLAT - Custom permissions
└── MyPerm.customPermission-meta.xml

roles/                         # FLAT - Roles
└── CEO.role-meta.xml

sharingRules/                  # FLAT - Sharing rules
└── ObjectName__c.sharingRules-meta.xml

queues/                        # FLAT - Queues
└── MyQueue.queue-meta.xml
```

### Configuration

```
labels/                        # FLAT - Custom labels (usually one file)
└── CustomLabels.labels-meta.xml

globalValueSets/               # FLAT - Global picklist values
└── MyPicklist.globalValueSet-meta.xml

customMetadata/                # FLAT - Custom metadata type records
└── MyType.MyRecord.md-meta.xml

remoteSiteSettings/            # FLAT - Remote site settings
└── MySite.remoteSite-meta.xml

reportTypes/                   # FLAT - Custom report types
└── MyReportType.reportType-meta.xml
```

### Assignment & Response Rules

```
assignmentRules/               # FLAT
└── ObjectName__c.assignmentRules-meta.xml

autoResponseRules/             # FLAT
└── ObjectName__c.autoResponseRules-meta.xml
```

### Static Resources

```
staticresources/               # Mixed: flat files or unzipped folders
├── MyResource.resource-meta.xml       # Always present
├── MyResource.css                     # Single-file resource
├── MyZippedResource.resource-meta.xml
└── MyZippedResource/                  # Unzipped folder resource
    ├── css/
    ├── js/
    └── images/
```

## Key Rules

1. **Every metadata file needs a -meta.xml companion** (except for metadata that IS the xml, like flows, layouts, objects)
2. **FLAT folder** = files directly in the folder, no subdirectories
3. **Subfolder pattern** = lwc/, aura/, objects/, staticresources/ — each item is its own directory
4. **Naming convention**: API names with `__c` suffix = custom, no suffix = standard
5. **`__mdt`** suffix = Custom Metadata Type objects (e.g., `Concepto_MP__mdt/`)
6. **`__e`** suffix = Platform Event objects
7. **`__b`** suffix = Big Object
8. **`__x`** suffix = External Object

## sfdx-project.json Reference

```json
{
  "packageDirectories": [
    { "path": "force-app", "default": true }
  ],
  "name": "my-project",
  "namespace": "",
  "sfdcLoginUrl": "https://login.salesforce.com",
  "sourceApiVersion": "64.0"
}
```

## Common CLI Commands

```bash
# Deploy specific metadata
sf project deploy start --source-dir force-app/main/default/lwc/myComp -o myOrg

# Deploy everything
sf project deploy start --source-dir force-app -o myOrg

# Retrieve metadata
sf project retrieve start --source-dir force-app -o myOrg

# Run Apex tests
sf apex run test --target-org myOrg --result-format human

# Execute anonymous Apex
sf apex run --file scripts/apex/myScript.apex --target-org myOrg
```
