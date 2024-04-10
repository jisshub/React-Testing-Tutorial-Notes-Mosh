# React Testing

For reference visit: https://github.com/mosh-hamedani/react-testing-starter

Github repo: https://github.com/jisshub/React-Testing-Tutorial-Notes-Mosh

For tutorial visit: https://www.youtube.com/watch?v=8Xwq35cPwYg&t=487s


## Install Vitest and vitest:ui

```bash
npm i vitest
npm i vitest:ui
```

## Run test script

```bash
npm run test
```

- Gives the test results in the terminal

## Run test script in UI

```bash
npm run test:ui
```

- Gives the test results in the browser.


## Setting React Testing Library

```bash
npm i @testing-library/react
```

## Testing React Components in an Environment with Vitest

- To test react component, we need to run the test in an environment that emulates the browser environment.
- We install jsdom as a dev dependency.

```bash
npm i -D jsdom@24.0.0
```

## Vitest config file

- Create vitest.config.ts file in root folder of the project.
- Configure the file to use jsdom as an environment.

```ts
import { defineConfig } from 'vitest/config';
export default defineConfig({
    test: {
        environment: 'jsdom',
    },
});
```

- Next install @testing-library/jest-dom.

```bash
npm i -D @testing-library/jest-dom
```


## Testing Rendering

*src\components\Greet.tsx*

```ts
const Greet = ({ name }: { name?: string }) => {
  if (name) return <h1>Hello {name}</h1>;

  return <button>Login</button>;
};

export default Greet;

```

*test\components\Greet.test.ts*
```ts
/* eslint-disable @typescript-eslint/no-unused-vars */
import { it, expect, describe } from "vitest";
import { render, screen } from "@testing-library/react";
import "@testing-library/jest-dom/vitest";
import Greet from "../../src/components/Greet";

describe("Greet", () => {
  it("should render Hello with the given name when the name is passed", () => {
    render(Greet({ name: "John" }));
    const heading = screen.getByRole("heading");
    expect(heading).toBeInTheDocument();
    expect(heading).toHaveTextContent(/hello john/i);
  });

  it("should render Login when the name is not passed", () => {
    render(Greet({ name: undefined }));
    const button = screen.getByRole("button");
    expect(button).toBeInTheDocument();
    expect(button).toHaveTextContent(/login/i);
  });
});
```

In the test suite, `getByRole('heading')` and `getByRole('button')` are queries provided by the Testing Library (specifically, `@testing-library/react`) used to select elements based on their [ARIA roles](https://www.w3.org/TR/wai-aria/#role_definitions). These queries help ensure that your components are accessible and function correctly for users with assistive technologies.

- `getByRole('heading')` is used to select the first element in the document that is recognized as a heading element (such as `<h1>`, `<h2>`, etc.). In the context of your test, it's used to find the `<h1>` element that renders "Hello {name}" when a name is passed to the Greet component.

- `getByRole('button')` is used to select the first element that is recognized as a button. In your test, it's used to find the `<button>` element that renders "Login" when no name is passed to the Greet  component.

These queries are part of the Testing Library's approach to encourage testing components in a way that reflects how users interact with the UI, focusing on accessibility and usability.

## Testing User Account

*src\components\UserAccount.tsx*

```ts
import { User } from "../entities";

const UserAccount = ({ user }: { user: User }) => {
  return (
    <>
      <h2>User Profile</h2>
      {user.isAdmin && <button>Edit</button>}
      <div>
        <strong>Name:</strong> {user.name}
      </div>
    </>
  );
};

export default UserAccount;

```

*test\components\UserAccount.test.ts*

```ts
import { it, expect, describe } from "vitest";
import { render, screen } from "@testing-library/react";
import UserAccount from "../../src/components/UserAccount";
import { User } from "../../src/entities";

describe("UserAccount", () => {
  it("should render user name", () => {
    // Setup
    const user: User = {
      id: 1,
      name: "John",
      isAdmin: true,
    };

    // Execution
    render(UserAccount({ user: user }));

    // Assertion
    expect(screen.getByText(user.name)).toBeInTheDocument();
  });

  it("should render the edit button if the user is admin", () => {
    // Setup
    const user: User = {
      id: 1,
      name: "John",
      isAdmin: true,
    };

    // Execution
    render(UserAccount({ user: user }));

    // Assertion
    const button = screen.getByRole("button");
    expect(button).toBeInTheDocument();
    expect(button).toHaveTextContent("Edit");
  });

  it("should not render the edit button if the user is not admin", () => {
    // Setup
    const user: User = {
      id: 1,
      name: "John",
      isAdmin: false,
    };

    // Execution
    render(UserAccount({ user: user }));

    // Assertion
    const button = screen.queryByRole("button");
    expect(button).not.toBeInTheDocument();
  });
});

```

Explanation
-----------
The choice between `getByRole('button')` and `queryByRole('button')` in these test cases is based on the expected outcome of rendering the UserAccount component under different conditions, specifically related to the presence or absence of the "Edit" button based on the isAdmin property of the user object.

1. **`getByRole('button')` in "should render the edit button if the user is admin" test case:**

   - getByRole is used when you expect an element to be present in the DOM. It throws an error if the element cannot be found, which immediately fails the test. This behavior is desirable when testing conditions that should result in the presence of an element, as it provides a clear and direct assertion that the element exists.

   - In the "should render the edit button if the user is admin" test case, the expectation is that the "Edit" button is rendered because `user.isAdmin` is `true`. Therefore, using `getByRole('button')` is appropriate because you expect the button to exist, and if it doesn't, the test should fail.


2. **`queryByRole('button')` in "should not render the edit button if the user is not admin" test case:**

    - queryByRole is used when you want to assert that an element is not present in the DOM. Unlike getByRole, it does not throw an error if the element cannot be found; instead, it returns `null`. This behavior is useful for verifying the absence of an element, as it allows you to assert that the result of queryByRole is `null` (or, in the case of Testing Library, that `not.toBeInTheDocument()`).

    - In the "should not render the edit button if the user is not admin" test case, the "Edit" button should not be rendered because `user.isAdmin` is `false`. Using `queryByRole('button')` is appropriate here because you're testing for the absence of the button, and the test should only fail if the button unexpectedly appears.

    - In summary, `getByRole`  is used when an element is expected to be present (and its absence should fail the test), while `queryByRole`is used when an element's absence is being verified (and its presence should fail the test).


## Testing Lists

*src\components\UserList.tsx*

```tsx
import { User } from "../entities";

const UserList = ({ users }: { users: User[] }) => {
  if (users.length === 0) return <p>No users available.</p>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>
          <a href={`/users/${user.id}`}>{user.name}</a>
        </li>
      ))}
    </ul>
  );
};

export default UserList;

```

test\components\UserList.test.ts
```ts
import "@testing-library/jest-dom";
import { it, expect, describe } from "vitest";
import { render, screen } from "@testing-library/react";
import UserList from "../../src/components/UserList";
import { User } from "../../src/entities";

describe("UserList", () => {
  it("should render no users when users array is empty", () => {
    const users: User[] = [
      {
        id: 1,
        name: "John",
      },
      {
        id: 2,
        name: "Teresa",
      },
    ];
    // Execution
    render(UserList({ users }));

    users.forEach((user) => {
      const link = screen.getByRole("link", { name: user.name });
      expect(link).toBeInTheDocument();
      expect(link.getAttribute("href")).toBe(`/users/${user.id}`);
    });

    // Assertions
    expect(screen.getByText(/no users/i)).toBeInTheDocument();
  });

  it("should render a list of users", () => {
    // Setup
    const users: User[] = [
      {
        id: 1,
        name: "John",
      },
      {
        id: 2,
        name: "Teresa",
      },
    ];
    // Execution
    render(UserList({ users: users }));

    // Assertions
    expect(screen.getByText(/john/i)).toBeInTheDocument();
  });
});


```

## Testing Product Image Gallery

### Component Code

*src\components\ProductImageGallery.tsx*
------------------------------------------

```tsx
const ProductImageGallery = ({ imageUrls }: { imageUrls: string[] }) => {
  if (imageUrls.length === 0) return null;

  return (
    <ul>
      {imageUrls.map((url) => (
        <li key={url}>
          <img src={url} />
        </li>
      ))}
    </ul>
  );
};

export default ProductImageGallery;

```

### Testing Code

*test\components\ProductImageGallery.test.ts*
------------------------------------------
```ts
import { it, expect, describe } from "vitest";
import "@testing-library/jest-dom";
import { render, screen } from "@testing-library/react";
import ProductImageGallery from "../../src/components/ProductImageGallery";

describe("ProductImageGallery", () => {
  it("should render no images when images array is empty", () => {
    const images: string[] = [];
    // Execution
    render(ProductImageGallery({ imageUrls: images }));
    // Assertions
    const list = screen.queryByRole("list");
    expect(list).not.toBeInTheDocument();
  });

  //   it should render images when images array is not empty
  it("should render images when images array is not empty", () => {
    const imageUrls: string[] = [
      "https://images.unsplash.com/photo-1494790108377-be9c29b29330?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=500&q=60",
      "https://images.unsplash.com/photo-1494790108377-be9c29b29330?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=500&q=60",
      "https://images.unsplash.com/photo-14947901083",
    ];
    // Execution
    render(ProductImageGallery({ imageUrls: imageUrls }));
    // Assertions
    const list = screen.getByRole("list");
    expect(list).toBeInTheDocument();

    const images = screen.getAllByRole("img");
    expect(images).toHaveLength(imageUrls.length);
    imageUrls.forEach((url, index) =>
      expect(images[index]).toHaveAttribute("src", url)
    );
  });
});
```

### Test Suite for ProductImageGallery Component


This test suite verifies the functionality of the ProductImageGallery component using the Testing Library and Vitest. The component is expected to render a list of images based on the imageUrls prop provided to it. The suite contains two primary test cases:


1. **Test Case: Rendering No Images When Array is Empty**

  This test verifies that the component correctly handles an empty array of image URLs. When no image URLs are provided, the component should not render a list element (`<ul>`).

  
   - **Setup**: An empty array of image URLs is created.
   
   - **Execution**: The ProductImageGallery component is rendered with this empty array.

   - **Assertion**: It is asserted that no list (`<ul>`) element is present in the document, indicating that the component does not render anything when there are no images to display.


2. **Test Case: Rendering Images When Array is Not Empty**

   This test checks the component's behavior when it is provided with an array of image URLs. The component should render a list of images, with each image's src attribute set to the corresponding URL from the array.

  
   - **Setup**: An array of image URLs is created.
   
   - **Execution**: The ProductImageGallery component is rendered with this array of image URLs.
   
   - **Assertion**:
   
     - It is first asserted that a list (`<ul>`) element is present in the document, indicating that the component renders a list when there are images to display.
    
     - Then, it is verified that the number of `<img>` elements matches the length of the image URLs array, ensuring that an image is rendered for each URL.

     - Finally, each `<img>` element's src attribute is checked against the corresponding URL in the array to ensure that images are correctly sourced from the provided URLs.
  
  These tests ensure that the ProductImageGallery  component behaves as expected, both when there are no images to display and when a list of images should be rendered. By covering these scenarios, the test suite helps maintain the component's reliability and functionality as the application evolves.


## Testing User Interaction

Time: 53: 15









