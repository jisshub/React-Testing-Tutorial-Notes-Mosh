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





