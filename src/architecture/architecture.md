# Architecture

The system mainly consists of a frontend website, a backend API and a database.
We will name all the technologies used in these components and also discuss how they are interconnected.

## Frontend

The frontend is built with **React**. We used an open-source admin template called Core UI. Here are some features we take advantage of from the template:

- Built with: React, Bootstrap, SASS
- Uses React hooks, components and context API
- Responsive and mobile-friendly design
- Lightweight - only 35KB when minified and gzipped
- Built-in charts: Chart.js, Google Charts, ECharts
- Built-in Routing using React Router

We chose to use a template instead of building the system from scratch because we want to get started and ship fast, and the system requirements are rather simple. Mainly CRUD functionalities are needed.

More on the frontend [here](./frontend.md).

&nbsp;

## Backend

The backend server is built with **NodeJS**. We used **Express** as the framework.
The reasons are we wanted to keep it simple, not only for myself but for future developers who might handle it. It's also fast to kick up a server with Express. And the server code is written in **TypeScript**.

More on the backend [here](./backend.md).

&nbsp;

## Database

We are utilizing **MySQL** as the database management system for this project with **PlanetScale** as the platform. We have chosen PlanetScale due to its ease of setup and use. We just want a simple credential-based access while still maintaining security. We trust PlanetScale to handle the security aspects of our database.

Additionally, PlanetScale is built on top of **Amazon RDS**, which provides us with the capability of auto-scaling in the future, should we need it. Although we are currently on the free tier, we are considering our future scaling needs.

&nbsp;

## Cloud

The React frontend app is hosted on **Netlify**. We chose Netlify due to its simplicity, as the frontend is a static site and deploying on Netlify is very easy. By connecting our GitHub repository and setting up some build configurations, the CI/CD is already working perfectly out of the box.

The NodeJS backend service runs on **AWS EC2** with CodePipeline managing the CI/CD. The process involves building a _Source_ artifact from GitHub then using that artifact and execute the steps defined in _appspec_ file via **CodeDeploy**.

We manually configured **Apache** on each server, exposed their IP addresses, mapped them to domain names using VirtualHosts, and enabled HTTPS connections utilizing **certbot** to establish the network architecture.

More on the deployment [here](../deployment/deployment.md).