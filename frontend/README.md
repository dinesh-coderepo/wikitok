# WikiTok

WikiTok is a TikTok-style interface for exploring Wikipedia articles. This project is built using React, TypeScript, and Vite, and it leverages Tailwind CSS for styling.

## Table of Contents

- [Setup](#setup)
- [Usage](#usage)
- [Key Functionalities](#key-functionalities)
- [Main Components](#main-components)
- [Running the Project](#running-the-project)
- [Prerequisites](#prerequisites)

## Setup

To set up the project, follow these steps:

1. Clone the repository:
   ```sh
   git clone https://github.com/dinesh-coderepo/wikitok.git
   cd wikitok/frontend
   ```

2. Install dependencies:
   ```sh
   npm install
   ```

## Usage

To start the development server, run:
```sh
npm run dev
```

To build the project for production, run:
```sh
npm run build
```

To preview the production build, run:
```sh
npm run preview
```

To lint the code, run:
```sh
npm run lint
```

## Key Functionalities

- **Infinite Scrolling**: The app fetches random Wikipedia articles and displays them in an infinite scrolling interface.
- **Article Sharing**: Users can share articles via the Web Share API or copy the link to the clipboard.
- **Responsive Design**: The app is designed to be responsive and works well on both desktop and mobile devices.

## Main Components

### App Component

The `App` component is the main component of the application. It handles the fetching of articles and manages the state of the application.

### WikiCard Component

The `WikiCard` component is responsible for displaying individual Wikipedia articles. It includes the article title, a brief extract, and a thumbnail image if available.

### useWikiArticles Hook

The `useWikiArticles` hook is a custom hook that handles the fetching of random Wikipedia articles. It manages the state of the articles and the loading state.

## Running the Project

To run the project locally, follow these steps:

1. Ensure you have Node.js and npm installed.
2. Clone the repository and navigate to the `frontend` directory.
3. Install the dependencies using `npm install`.
4. Start the development server using `npm run dev`.

## Prerequisites

- Node.js (version 14 or higher)
- npm (version 6 or higher)

## Expanding the ESLint configuration

If you are developing a production application, we recommend updating the configuration to enable type aware lint rules:

- Configure the top-level `parserOptions` property like this:

```js
export default tseslint.config({
  languageOptions: {
    // other options...
    parserOptions: {
      project: ['./tsconfig.node.json', './tsconfig.app.json'],
      tsconfigRootDir: import.meta.dirname,
    },
  },
})
```

- Replace `tseslint.configs.recommended` to `tseslint.configs.recommendedTypeChecked` or `tseslint.configs.strictTypeChecked`
- Optionally add `...tseslint.configs.stylisticTypeChecked`
- Install [eslint-plugin-react](https://github.com/jsx-eslint/eslint-plugin-react) and update the config:

```js
// eslint.config.js
import react from 'eslint-plugin-react'

export default tseslint.config({
  // Set the react version
  settings: { react: { version: '18.3' } },
  plugins: {
    // Add the react plugin
    react,
  },
  rules: {
    // other rules...
    // Enable its recommended rules
    ...react.configs.recommended.rules,
    ...react.configs['jsx-runtime'].rules,
  },
})
```
