# Editorial Manager System Guide

## DOM Structure

Editorial Manager (EM) uses iframe nesting. The main content is inside:
```javascript
document.querySelector('iframe[name="content"]').contentDocument
```

All interactions with form elements must target this iframe's contentDocument.

## File Upload

EM file upload requires special handling due to iframe context:

### Method 1: FileChooser Event (Recommended)

```javascript
// Register filechooser handler before clicking upload button
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const fileInput = iframe.querySelector('input[type="file"]');

// Set files programmatically
const dataTransfer = new DataTransfer();
const file = new File([fileContent], 'filename.docx', { type: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document' });
dataTransfer.items.add(file);
fileInput.files = dataTransfer.files;

// Trigger change event
fileInput.dispatchEvent(new Event('change', { bubbles: true }));
```

### Method 2: Click + FileChooser (For Playwright)

```javascript
// In playwright-cli context:
// 1. Register filechooser handler
// 2. Click upload button
// 3. Handler receives file path and sets it

// Example playwright-cli eval:
(() => {
  const iframe = document.querySelector('iframe[name="content"]').contentDocument;
  const fileInput = iframe.querySelector('input[type="file"]');
  // File will be set via playwright's fileChooser event
  fileInput.click();
})();
```

### File Type Selection

After file is selected, choose the correct file type from dropdown:
```javascript
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const fileTypeSelect = iframe.querySelector('select[name="fileType"]');
fileTypeSelect.value = 'Manuscript'; // or 'Cover Letter', 'Figure', etc.
fileTypeSelect.dispatchEvent(new Event('change', { bubbles: true }));
```

## CKEditor Rich Text Fields

EM uses CKEditor for abstract and other rich text fields.

### Setting Content

```javascript
// Find the CKEditor instance
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const editorInstance = iframe.CKEDITOR.instances['abstract']; // or other field name

// Set HTML content
editorInstance.setData('<p>Your abstract text here</p>');
```

### Getting Content

```javascript
const content = iframe.CKEDITOR.instances['abstract'].getData();
```

### Available Instances

List all CKEditor instances:
```javascript
Object.keys(iframe.CKEDITOR.instances);
```

## Author Reordering

### Method 1: Drag and Drop (If Supported)

Some EM instances support HTML5 drag and drop:
```javascript
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const authors = iframe.querySelectorAll('.author-item');

// Simulate drag from index 1 to index 0
const source = authors[1];
const target = authors[0];

const dragStartEvent = new DragEvent('dragstart', { bubbles: true, cancelable: true });
const dropEvent = new DragEvent('drop', { bubbles: true, cancelable: true });
const dragEndEvent = new DragEvent('dragend', { bubbles: true, cancelable: true });

source.dispatchEvent(dragStartEvent);
target.dispatchEvent(dropEvent);
source.dispatchEvent(dragEndEvent);
```

### Method 2: DOM Manipulation (jQuery UI Sortable)

Most EM instances use jQuery UI sortable:
```javascript
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const authors = iframe.querySelectorAll('.author-item');

// Move author from index 1 to index 0
const source = authors[1];
const target = authors[0];

// Insert before target
target.parentNode.insertBefore(source, target);

// Refresh sortable
iframe.$(iframe).find('.author-list').sortable('refresh');
```

### Method 3: Button Controls

Some EM instances provide up/down buttons:
```javascript
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const moveUpBtn = iframe.querySelector('.author-item:nth-child(2) .move-up-btn');
moveUpBtn.click();
```

## Handling "Institution Unverified" Warnings

When EM cannot verify an institution from its database, it shows a warning dialog.

### Detection

```javascript
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const dialog = iframe.querySelector('.ui-dialog');
const dialogText = dialog?.textContent || '';

if (dialogText.includes('Institution Unverified') || dialogText.includes('institution could not be verified')) {
  // Handle warning
}
```

### Resolution

Simply click OK to continue:
```javascript
const okButton = iframe.querySelector('.ui-dialog-buttonset button');
okButton.click();
```

## Form Field Interaction

### Text Inputs

```javascript
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const input = iframe.querySelector('input[name="fieldName"]');
input.value = 'New value';
input.dispatchEvent(new Event('input', { bubbles: true }));
input.dispatchEvent(new Event('change', { bubbles: true }));
```

### Dropdowns

```javascript
const select = iframe.querySelector('select[name="fieldName"]');
select.value = 'optionValue';
select.dispatchEvent(new Event('change', { bubbles: true }));
```

### Checkboxes

```javascript
const checkbox = iframe.querySelector('input[type="checkbox"][name="fieldName"]');
checkbox.checked = true;
checkbox.dispatchEvent(new Event('change', { bubbles: true }));
```

### Radio Buttons

```javascript
const radio = iframe.querySelector('input[type="radio"][name="fieldName"][value="optionValue"]');
radio.checked = true;
radio.dispatchEvent(new Event('change', { bubbles: true }));
```

## Saving and Navigation

### Save Button

Most tabs have a Save button:
```javascript
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const saveBtn = iframe.querySelector('button[name="save"], input[type="submit"][value="Save"]');
saveBtn.click();
```

### Next/Continue Button

```javascript
const nextBtn = iframe.querySelector('button[name="next"], input[type="submit"][value="Next"]');
nextBtn.click();
```

## Popup/Dialog Handling

### Confirmation Dialogs

```javascript
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const dialog = iframe.querySelector('.ui-dialog:not([style*="display: none"])');

if (dialog) {
  const confirmBtn = dialog.querySelector('.ui-dialog-buttonset button:first-child');
  confirmBtn.click();
}
```

### Error Messages

```javascript
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const errorMsg = iframe.querySelector('.error-message, .ui-state-error');

if (errorMsg) {
  console.log('Error:', errorMsg.textContent);
}
```

## PDF Generation

### Trigger PDF Build

```javascript
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const buildBtn = iframe.querySelector('button[name="buildPDF"], input[type="submit"][value="Build PDF"]');
buildBtn.click();
```

### Check PDF Status

```javascript
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const status = iframe.querySelector('.pdf-status, .build-status');
const isReady = status?.textContent.includes('ready') || status?.textContent.includes('complete');
```

### Download PDF

```javascript
const iframe = document.querySelector('iframe[name="content"]').contentDocument;
const downloadLink = iframe.querySelector('a[href*="downloadPDF"], a.download-pdf');
const pdfUrl = downloadLink.href;
```

## Common Selectors

| Element | Selector |
|---------|----------|
| Content iframe | `iframe[name="content"]` |
| Save button | `button[name="save"], input[value="Save"]` |
| Next button | `button[name="next"], input[value="Next"]` |
| File input | `input[type="file"]` |
| Author list | `.author-list, .authors-container` |
| Author item | `.author-item, .author-row` |
| Dialog | `.ui-dialog:not([style*="display: none"])` |
| Error message | `.error-message, .ui-state-error` |

## Troubleshooting

### Element Not Found

If elements are not found, verify:
1. You're targeting the iframe's contentDocument
2. The page has fully loaded (wait for network idle)
3. The element selector is correct (inspect in browser dev tools)

### Changes Not Saving

After setting values programmatically:
1. Dispatch 'input' and 'change' events
2. Ensure the form field is not disabled or read-only
3. Check for validation errors

### File Upload Fails

Common issues:
1. File size exceeds limit
2. File format not allowed
3. File input not properly initialized
4. Missing file type selection

### Author Reorder Fails

If drag-and-drop doesn't work:
1. Check if jQuery UI sortable is initialized
2. Try DOM manipulation method instead
3. Look for up/down buttons as alternative
