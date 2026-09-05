# EvoSynth Documentation Hub

This repository contains the official documentation for EvoSynth, an advanced AutoML Meta-Optimizer that unifies evolutionary algorithms (GA, PSO, DE, CMA-ES) and a Multi-Armed Bandit (UCB1) over a continuous Universal Latent Space.

The documentation is built using VitePress and provides comprehensive guides, mathematical breakdowns of the core algorithms, and API references.

## Project Structure

- `.vitepress/`: Contains the core configuration (`config.mts`) and custom theme styling for the documentation site.
- `guide/`: Getting started instructions, installation guide, and high-level architecture overview.
- `concepts/`: Deep dives into the Meta-Optimizer, Universal Latent Space, UCB1 Bandit, and Composite Scoring Engine.
- `algorithms/`: Detailed explanations and mathematical formulations of the worker optimizers (Genetic Algorithm, Particle Swarm, Differential Evolution, CMA-ES).
- `dashboard/`: Details regarding the React telemetry dashboard and the Server-Sent Events (SSE) streaming protocol.
- `developer/`: Guides on how to extend the framework, including adding custom evaluators or building new optimizers.
- `reference/`: API documentation for the core classes and modules.

## Local Development

To run the documentation site locally, ensure you have Node.js installed, then execute the following commands:

1. Install dependencies:
```bash
npm install
```

2. Start the local development server:
```bash
npm run docs:dev
```

3. Open your browser and navigate to the provided localhost URL (typically `http://localhost:5173`).

## Building for Production

To build the static HTML files for deployment:

```bash
npm run docs:build
```

The generated static files will be located in the `.vitepress/dist` directory. These files can be deployed to any static hosting service, such as GitHub Pages, Vercel, or Netlify.

## Contributing

When contributing to this documentation, please ensure that all new files are appropriately linked in the `.vitepress/config.mts` sidebar array to remain visible on the deployed site.
