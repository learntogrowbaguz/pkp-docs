---
book: dev-documentation
version: 3.4
title: Submission Files - Technical Documentation - OJS|OMP|OPS
---

# Submission Files

> Read and understand how [Files](./utilities-files) are handled before continuing.
{:.tip}

Submission files are used to track the progress of a file through the editorial workflow. For example, a file might be uploaded by the author during submission, promoted to the review stage by an editor, revised again by the author, and downloaded by a copyeditor.

Each `SubmissionFile` represents a file at one of these file stages. For example, when an editor promotes a file from the Submission stage to the Review stage, there are two `SubmissionFile` objects which refer to the same file.

| SubmissionFile::submission_file_id | file_id | file_stage                                  |
| ---------------------------------- | ------- | ------------------------------------------- |
| 1                                  | 82      | SubmissionFile::SUBMISSION_FILE_SUBMISSION  |
| 2                                  | 82      | SubmissionFile::SUBMISSION_FILE_REVIEW_FILE |

Submission files can be revised. For example, the file in the Review stage may need to be anonymized before it can be sent for review. When the editor uploads a modified copy as a revision, the `file_id` is changed but a new `SubmissionFile` is not created.

| SubmissionFile::submission_file_id | file_id | file_stage                                  |
| ---------------------------------- | ------- | ------------------------------------------- |
| 1                                  | 82      | SubmissionFile::SUBMISSION_FILE_SUBMISSION  |
| 2                                  | 83      | SubmissionFile::SUBMISSION_FILE_REVIEW_FILE |
| 2                                  | 84      | SUBMISSION_FILE_REVIEW_FILE                 |

Editors and assistants can access all revisions of a file in the submission file's activity log.

## File Stages

All submission files are assigned to one file stage. File stages overlap with the submission workflow stages, but each workflow stage can have more than one file stage. For example, the Review workflow stage has one file stage for files to be sent to reviewers and another file stage for revisions uploaded by the author.

Most file stages correspond to a list of files in the submission workflow, such as the Review Files, Copyedited Files, or Production Ready Files. Other file stages are used to identify submission files that are attached to discussions, review assignments and galleys or publication formats.

| File Stage                                                 | Description                                                                                                                                                                                                                   |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SubmissionFile::SUBMISSION_FILE_SUBMISSION`               | Files uploaded during submission.                                                                                                                                                                                             |
| `SubmissionFile::SUBMISSION_FILE_REVIEW_FILE`              | Files to be sent for peer review.                                                                                                                                                                                             |
| `SubmissionFile::SUBMISSION_FILE_REVIEW_ATTACHMENT`        | Files uploaded by a reviewer.                                                                                                                                                                                                 |
| `SubmissionFile::SUBMISSION_FILE_ATTACHMENT`               | Files attached by an editor to the email requesting revisions. These files are not displayed in the author dashboard.                                                                                                         |
| `SubmissionFile::SUBMISSION_FILE_REVIEW_REVISION`          | Revisions uploaded by the author after peer review.                                                                                                                                                                           |
| `SubmissionFile::SUBMISSION_FILE_INTERNAL_REVIEW_FILE`     | Files to be sent for internal review. Only used in OMP.                                                                                                                                                                       |
| `SubmissionFile::SUBMISSION_FILE_INTERNAL_REVIEW_REVISION` | Revisions uploaded by the author after peer review. Only used in OMP.                                                                                                                                                         |
| `SubmissionFile::SUBMISSION_FILE_FINAL`                    | Files to be copyedited.                                                                                                                                                                                                       |
| `SubmissionFile::SUBMISSION_FILE_COPYEDIT`                 | Files that have been copyedited.                                                                                                                                                                                              |
| `SubmissionFile::SUBMISSION_FILE_PRODUCTION_READY`         | Files ready to be typeset. For example, a file that is ready to be converted to PDF.                                                                                                                                          |
| `SubmissionFile::SUBMISSION_FILE_PROOF`                    | Files attached to a galley (OJS/OPS) or publication format (OMP).                                                                                                                                                             |
| `SubmissionFile::SUBMISSION_FILE_DEPENDENT`                | Files that are never downloaded directly but are attached to another file. For example, a CSS file uploaded as a dependent file to a HTML file. Dependent files are uploaded separately when an HTML or XML file is detected. |
| `SubmissionFile::SUBMISSION_FILE_QUERY`                    | Files uploaded to a discussion.                                                                                                                                                                                               |
| `SubmissionFile::SUBMISSION_FILE_NOTE`                     | This file stage is not used.                                                                                                                                                                                                  |

Files assigned to the review file or revision stages must be associated with a review round.

```php
use PKP\submissionFile\SubmissionFile;

$submissionFile->setData('fileStage', SubmissionFile::SUBMISSION_FILE_REVIEW_FILE);
$submissionFile->setData('assocType', ASSOC_TYPE_REVIEW_ROUND);
$submissionFile->setData('assocId', $reviewRoundId);
```

Files assigned to the `SubmissionFile::SUBMISSION_FILE_QUERY` file stage must be associated with a query note.

```php
use PKP\submissionFile\SubmissionFile;

$submissionFile->setData('fileStage', SubmissionFile::SUBMISSION_FILE_QUERY);
$submissionFile->setData('assocType', ASSOC_TYPE_NOTE);
$submissionFile->setData('assocId', $noteId);
```

## Submission File Repository

Use the `Repository` to add, edit and delete submission files. This ensures that event logs are kept to track revisions, identify the uploader of a file, and update pending tasks.

Use the [File Service](./utilities-files) to create a file. Then assign the `fileId` to a `SubmissionFile`.

```php
use APP\core\Application;
use APP\core\Services;
use APP\facades\Repo;

$fileId = Services::get('file')->add($source, $destination);
Repo::submissionFile()->edit(
	$submissionFile,
	[
		'fileId' => $fileId,
		'uploaderUserId' => Application::get()->getRequest()->getUser()->getId(),
	]
);
```

Submission files can not be created without required properties.

```php
use APP\facades\Repo;
use PKP\submissionFile\SubmissionFile;

$submissionFile = Repo::submissionFile()->newDataObject([
	'fileStage' => SubmissionFile::SUBMISSION_FILE_REVIEW_FILE,
	'fileId' => $fileId,
	'name' => [
		$primaryLocale => 'my-filename.txt',
	],
	'submissionId' => $submissionId,
	'uploaderUserId' => $userId,
]);
```

Validate the submission file before it is added or edited.

```php
use APP\facades\Repo;
use PKP\submissionFile\SubmissionFile;

$params = [
	'fileStage' => SubmissionFile::SUBMISSION_FILE_REVIEW_FILE,
	'fileId' => $fileId,
	'name' => [
		$primaryLocale => 'my-filename.txt',
	],
	'submissionId' => $submissionId,
	'uploaderUserId' => $userId,
];

$errors = Repo::submissionFile()->validate(null, $params, $allowedLocales, $primaryLocale);

if (empty($errors)) {
	$submissionFile = Repo::submissionFile()->newDataObject($params);
	$id = Repo::submissionFile()->add($submissionFile);
}
```

## Component Types

Submission files should be assigned a `genreId`. This matches one of the submission component types, such as Article Text, Data Set or Book Monograph.

```php
$submissionFile->setData('genreId', 1);
```

## File Access

Access to a submission file is granted based on the user's assignment to a submission. For example, an author should not be able to access files uploaded by reviewers. A copyeditor should only be able to access files in the copyediting stage.

Use the `getAssignedFileStages()` method to determine what file stages can be accessed by a manager, subeditor, assistant or author.

```php
use APP\facades\Repo;
use PKP\submissionFile\SubmissionFile;

$stageAssignments = $this->getAuthorizedContextObject(ASSOC_TYPE_ACCESSIBLE_WORKFLOW_STAGES);
$assignedFileStages = Repo::submissionFile()->getAssignedFileStages($stageAssignments, SubmissionFile::SUBMISSION_FILE_ACCESS_READ);
```

Use the `ReviewFilesDAO` to check if a reviewer can access a file.

```php
use PKP\db\DAORegistry;

$reviewFilesDao = DAORegistry::getDAO('ReviewFilesDAO'); /* @var $reviewFilesDao ReviewFilesDAO */
$reviewerCanAccess = $reviewFilesDao->check($reviewAssignment->getId(), $submissionFile->getId());
```

Use the `QueryDAO` to check if a user can access a discussion file.

```php
use APP\core\Application;
use PKP\db\DAORegistry;
use PKP\submissionFile\SubmissionFile;

if (
	$submissionFile->getData('fileStage') === SubmissionFile::SUBMISSION_FILE_QUERY &&
	$submissionFile->getData('assocType') === ASSOC_TYPE_NOTE
) {
	$queryDao = DAORegistry::getDAO('QueryDAO'); /* @var $queryDao QueryDAO */
	$noteDao = DAORegistry::getDAO('NoteDAO'); /* @var $noteDao NoteDAO */

	$user = Application::get()->getRequest()->getUser();
	$note = $noteDao->getById($submissionFile->getData('assocId'));

	if ($queryDao->getParticipantIds($note->getAssocId(), $user->getId())) {
		// User can access this submission file
	}
}
```

The following table describes when users are granted access to submission files.

| File Stage                                                             | Manager | Subeditor<sup>1</sup> | Assistant<sup>1</sup> | Author          | Reviewer        |
| ---------------------------------------------------------------------- | ------- | --------------------- | --------------------- | --------------- | --------------- |
| `SubmissionFile::SUBMISSION_FILE_SUBMISSION`                           | ✔       | ✔                     | ✔                     | ✔<sup>2</sup>   |                 |
| `SubmissionFile::SUBMISSION_FILE_REVIEW_FILE`                          | ✔       | ✔                     | ✔                     | ✔<sup>4,8</sup> | ✔<sup>7,8</sup> |
| `SubmissionFile::SUBMISSION_FILE_REVIEW_ATTACHMENT`                    | ✔       | ✔                     | ✔                     | ✔<sup>4</sup>   | ✔               |
| `SubmissionFile::SUBMISSION_FILE_ATTACHMENT`                           | ✔       | ✔                     | ✔                     | ✔               |                 |
| `SubmissionFile::SUBMISSION_FILE_REVIEW_REVISION`                      | ✔       | ✔                     | ✔                     | ✔<sup>3</sup>   |                 |
| `SubmissionFile::SUBMISSION_FILE_INTERNAL_REVIEW_FILE`<sup>6</sup>     | ✔       | ✔                     | ✔                     | ✔<sup>4,8</sup> | ✔<sup>7,8</sup> |
| `SubmissionFile::SUBMISSION_FILE_INTERNAL_REVIEW_REVISION`<sup>6</sup> | ✔       | ✔                     | ✔                     | ✔<sup>3</sup>   |                 |
| `SubmissionFile::SUBMISSION_FILE_FINAL`                                | ✔       | ✔                     | ✔                     |                 |                 |
| `SubmissionFile::SUBMISSION_FILE_COPYEDIT`                             | ✔       | ✔                     | ✔                     | ✔<sup>8</sup>   |                 |
| `SubmissionFile::SUBMISSION_FILE_PRODUCTION_READY`                     | ✔       | ✔                     | ✔                     |                 |                 |
| `SubmissionFile::SUBMISSION_FILE_PROOF`                                | ✔       | ✔                     | ✔                     | ✔<sup>8</sup>   |                 |
| `SubmissionFile::SUBMISSION_FILE_DEPENDENT`<sup>9</sup>                | ?       | ?                     | ?                     | ?               | ?               |
| `SubmissionFile::SUBMISSION_FILE_QUERY`<sup>5</sup>                    | ?       | ?                     | ?                     | ?               | ?               |

1. Access is granted when the user group is assigned to the related workflow stage.
2. Access is granted before submission is complete.
3. Access is granted after revisions are requested.
4. Access is granted when file has been shared with author during editor decision. The `viewable` column in the `submission_files` table is set to true.
5. Access is granted when the user is assigned to the correct query. Users can only delete files attached to their own query notes.
6. Only used in OMP.
7. Access is granted when file has been shared with the review assignment. An entry is added to the `review_files` table.
8. Read access only.
9. Access to dependent files is granted if the user has access to the parent file.


