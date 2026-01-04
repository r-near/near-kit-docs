# Mintlify documentation

## Working relationship
- You can push back on ideas - this can lead to better documentation. Cite sources and explain your reasoning when you do so
- ALWAYS ask for clarification rather than making assumptions
- NEVER lie, guess, or make up anything

## Project context
- Format: MDX files with YAML frontmatter
- Config: docs.json for navigation, theme, settings
- Components: Mintlify components
- This is documentation for `near-kit`, a TypeScript library for interacting with the NEAR blockchain

## Content strategy
- Document just enough for user success - not too much, not too little
- Prioritize accuracy and usability
- Make content evergreen when possible
- Search for existing content before adding anything new. Avoid duplication unless it is done for a strategic reason
- Check existing patterns for consistency
- Start by making the smallest reasonable changes

## docs.json

- Refer to the [docs.json schema](https://mintlify.com/docs.json) when building the docs.json file and site navigation

## Frontmatter requirements for pages
- title: Clear, descriptive page title
- description: Concise summary for SEO/navigation

## Writing standards
- Second-person voice ("you")
- Prerequisites at start of procedural content
- Test all code examples before publishing
- Match style and formatting of existing pages
- Include both basic and advanced use cases
- Language tags on all code blocks
- Alt text on all images
- Relative paths for internal links

## Mintlify components

Use these components when appropriate:

### Callouts
```mdx
<Note>Informational note</Note>
<Warning>Warning message</Warning>
<Tip>Helpful tip</Tip>
<Info>Additional info</Info>
```

### Layout
```mdx
<CardGroup cols={2}> - Navigation cards
<Steps>, <Step> - Numbered tutorials
<Tabs>, <Tab> - Tabbed content (use for before/after comparisons, multi-environment examples)
<AccordionGroup>, <Accordion> - Collapsible sections
<CodeGroup> - Multi-language code blocks
```

### API Documentation
```mdx
<ResponseField name="param" type="string" required>
  Description of the parameter
</ResponseField>
```

## Git workflow
- NEVER use --no-verify when committing
- Ask how to handle uncommitted changes before starting
- Create a new branch when no clear branch exists for changes
- Commit frequently throughout development
- NEVER skip or disable pre-commit hooks

## Do not
- Skip frontmatter on any MDX file
- Use absolute URLs for internal links
- Include untested code examples
- Make assumptions - always ask for clarification

## Deployment
The site deploys automatically via Mintlify when changes are pushed to the `main` branch. No CI/CD workflow is needed.

## Local development

```bash
# Preview locally at http://localhost:3000
npx mint dev
```
