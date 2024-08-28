# TDD AWS Amplify Next App - Step 15

## Add Note Image
The next story is `Note Image`.  Be sure to move it into `In Progress` once you start working on it.

### Why: User Story

```
As a team member
I want to add an image to a note
So that I provide visual context
```

### What: User Acceptance Criteria

```
Given a valid name and description are provided
When an image is uploaded successfully
And the note is saved successfully
Then the image is displayed under the name and description in the note list
```

```
Given a valid name and description are provided
When no image is added
Then the note is diplayed without an image
```

```
Given a valid name and description are provided
When an image is not uploaded successfully
And the note is saved anyway
Then the note is diplayed without an image
```

```
Given a non-image file is selected
When the user attempts to upload the file
Then they are prevented from uploading the file
```

```
Given a user wishes to select more than one file
When they attempt to select more than one file
Then they are prevented from selecting more than one file
```

- Run `npm i @aws-amplify/ui-react-storage`

- Create a new folder `storage` under the `amplify` directory
- In this new `storage` folder create a file named `resource.ts` and add the following content

```js
import { defineStorage } from "@aws-amplify/backend";

export const storage = defineStorage({
  name: "note-image",
  access: (allow) => ({
    "images/*": [allow.authenticated.to(["read", "write", "delete"])],
  }),
});
```

-Update `amplify/backend.ts` with the following

```js
import { defineBackend } from '@aws-amplify/backend';
import { auth } from './auth/resource';
import { data } from './data/resource';
import { storage } from './storage/resource';

defineBackend({
  auth,
  storage,
  data,
});
```

- Now let's add a [StorageManager](https://ui.docs.amplify.aws/react/connected-components/storage/storagemanager) component to our `NoteForm` component

```js
...
import { StorageManager } from '@aws-amplify/ui-react-storage';
import "@aws-amplify/ui-react/styles.css";


export default function NoteForm() {
  ...

  return (
    <div data-testid="note-form">
      ...
      <StorageManager
          acceptedFileTypes={['image/*']}
          path={({ identityId }) => `protected/${identityId}/`}
          maxFileCount={1}
          maxFileSize={500000}
          isResumable
        />
      <button
        data-testid="note-form-submit"
        type="button"
        onClick={createNote}
      >
        Create Note
      </button>
    </div>
  );
}
```
- By adding the `path={({ identityId }) => `protected/${identityId}/`}` callback the storage manager will upload an image to [private or protected S3 bucket](https://ui.docs.amplify.aws/react/connected-components/storage/storagemanager#private-or-protected-buckets)
- By adding `acceptedFileTypes={['image/*']}` only images file types can be uploaded.  This helps prevent uploading files that are not able to be displayed as an image.
- By adding `maxFileCount={1}` the number of uploaded files are limited to one
- By adding `maxFileSize={500000}` the file size is limited to 500KB.  This is critical for preventing costly mistakes or even attacks.

In order to save the path to this file we need to add an `onUploadSucces` callback function.

```js
...
export default function NoteForm() {
  const [formData, setFormData] = useState({ name: "", description: "", imageLocation: ""});
  ...
  const handleFileSuccess = ({ key }: { key?: string }) => {
    if (key) {
      setFormData(formData..., imageLocation: key);
    } 
  };
  ...
  return (
    <div data-testid="note-form">
      ...
      <StorageManager
          ...
         onChange={handleChange} 
        />
      ...
    </div>
  );
}
```

In order to save this you will need to update your data model with the following in `amplify/data/resource.ts`.

```js
const schema = a.schema({
  Note: a
    .model({
      name: a.string(),
      description: a.string(),
      imageLocation: a.string()
    })
    .authorization((allow) => [allow.publicApiKey()]),
});
```

Now let's display the image in the `NoteList` component.

```js
...
import { StorageImage } from '@aws-amplify/ui-react-storage';

export default function NoteList() {
  ...

  return (
    <div data-testid="note-list">
      {notes.map((note, index) => (
        <div key={index}>
          <p data-testid={`test-name-${index}`}>{note.name}</p>
          <p data-testid={`test-description-${index}`}>{note.description}</p>
          <StorageImage
            alt="note image ${index}"
            path={({ identityId }) => `protected/${identityId}/{note.imageLocatio}`}
          />
          <button
            type="button"
            data-testid={`test-delete-button-${index}`}
            onClick={() => deleteNote(note.id)}>
            Delete note
          </button>
        </div>
      ))}
    </div>
  );
}
```

- Run all the tests
- Green
- Commit

Demonstrate this new ability to delete notes to the customer.  If they accept this story then move it to `Done`.  If they request any changes, leave it `In Progress` and keep working on it.  Once they accept the story, move it to `Done` and move the next highest story from the `Todo` column to the `In Progress` column.

[<kbd> Previous Step </kbd>](https://github.com/pairing4good/tdd-next-amplify-gen2-tutorial/tree/015-step)&ensp;&ensp;&ensp;&ensp;[<kbd> Next Step </kbd>](https://github.com/pairing4good/tdd-next-amplify-gen2-tutorial/tree/017-step)
