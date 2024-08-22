# TDD AWS Amplify Next App - Step 12

## Add Authentication

- Run `npm install @aws-amplify/ui-react`

- Add the following just under the imports in the `src/app/layout.tsx` file

```js
"use client";

import Header from "./header";
import { Amplify } from "aws-amplify";
import outputs from "../../amplify_outputs.json";
import { Authenticator } from "@aws-amplify/ui-react";
import "@aws-amplify/ui-react/styles.css";

Amplify.configure(outputs);

export default function RootLayout({
  children,
}: Readonly<{ children: React.ReactNode }>) {
  return (
    <html lang="en">
      <body>
        <Header />
        <Authenticator>
          {({ signOut, user }) => (
            <>
              <button onClick={signOut}>Sign out</button>
              <main>{children}</main>
            </>
          )}
        </Authenticator>
      </body>
    </html>
  );
}
```

- Run `npm run dev`

- Open http://localhost:3000
- Click the `Create account` link
- Create and Verify your new account
- Login to your App

- Run all your tests
- While the non-UI tests pass, the Cypress tests are **Red**.

### Cypress Login

The Cypress tests now need to log in to the notes app.

- Run `npm install cypress-localstorage-commands`
- Add the following to the `cypress/support/commands.js` file

```js
import 'cypress-localstorage-commands';
import { Amplify } from "aws-amplify";
import { fetchAuthSession, signIn } from "aws-amplify/auth";
import outputs from "../../amplify_outputs.json";

Amplify.configure(outputs);


Cypress.Commands.add('signIn', () => { 
    const username = "[your-email]";
    const password = "[your-password]"
    const userPoolClientId = outputs.auth.user_pool_client_id;
    const keyPrefix = `CognitoIdentityServiceProvider.${userPoolClientId}`;
    
    cy.wrap(signIn({ username, password}))
        .then((_signInOutput) => fetchAuthSession())
        .then((authSession) => {
            const idToken = authSession.tokens?.idToken?.toString() || ''
            const accessToken = authSession.tokens?.accessToken.toString() || ''
            
            cy.setLocalStorage(`${keyPrefix}.${username}.accessToken`, accessToken);
            cy.setLocalStorage(`${keyPrefix}.${username}.idToken`, idToken);
            cy.setLocalStorage(`CognitoIdentityServiceProvider.${userPoolClientId}.LastAuthUser`, username);
            cy.saveLocalStorage();
        });
 })

declare global {
  namespace Cypress {
    interface Chainable {
        signIn(): Chainable<void>
    }
  }
}
```
> The [asynchronous nature of Cypress](https://learn.cypress.io/cypress-fundamentals/understanding-the-asynchronous-nature-of-cypress) is "is arguably one of the most crucial Cypress concepts that you need to understand. How Cypress handles things asynchronously is often misunderstood by developers and can lead to issues and confusion later on, especially when trying to debug your tests.

- [cy.wrap](https://docs.cypress.io/api/commands/wrap) is used to ensure that `signIn` is completed before running the rest of the tests.
- Replace `[your-email]` with the email you used to create your account
- Replace `[your-password]` with the password you used to create your account
  
- Add the following setups and teardowns to `cypress/integration/note.cy.js`

```js
before(() => {
  cy.signIn();
});

beforeEach(() => {
  cy.restoreLocalStorage();
  cy.visit("/");
});

after(() => {
  cy.clearLocalStorage();
});

afterEach(() => {
  cy.clearLocalStorage('notes');
  cy.saveLocalStorage();
});
```

- Rerun all of your tests.
- Green!
- Commit

[<kbd> Previous Step </kbd>](https://github.com/pairing4good/tdd-next-amplify-gen2-tutorial/tree/011-step)&ensp;&ensp;&ensp;&ensp;[<kbd> Next Step </kbd>](https://github.com/pairing4good/tdd-next-amplify-gen2-tutorial/tree/013-step)
