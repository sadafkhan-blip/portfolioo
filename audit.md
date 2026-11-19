# Portfolio implementation audit — Sadaf Khan

## Scope reviewed

This audit is based on the supplied portfolio route source (`createFileRoute("/")`) only. It references configuration, UI components, an image asset, and a server-side contact submission function that were not included in the supplied source excerpt. Those dependent files and the running site could not be verified.

## What the supplied route implements

- Five main navigation destinations: Home, About, Skills, Projects, and Contact.
- Responsive navigation state, including a mobile menu toggle.
- Animated role text cycling through Generative AI, Aspiring DevOps Engineer, and Aspiring MLOps Engineer.
- Scroll-based active-section tracking and reveal animations using `IntersectionObserver`.
- About/education content and interest chips.
- Skill cards generated from `skillGroups`.
- Project cards generated from `portfolioConfig.socialProjects`.
- Contact details and profile links generated from `portfolioConfig`.
- A contact form that validates with `contactSubmissionSchema` and calls `submitPortfolioContact` through `useServerFn`.
- SEO metadata declared in the TanStack route `head` option.
- Resume behavior: uses `portfolioConfig.resumePath` when present; otherwise provides a mailto-based resume-request link.

## Findings and follow-up checks

### 1. Confirm the framework document entry

**Observation:** The supplied file is a TanStack Router / TanStack Start route, not a standalone Vite React entry file. It uses `createFileRoute` and `useServerFn`.

**Action:** Keep the framework's existing root route, document shell, server entry, and routing setup. Do not create a second React root or blindly add a `/src/main.tsx` entry point. The accompanying `index.html` is a metadata/document-shell reference; use it only if the actual project setup expects a static HTML entry. In a TanStack Start app, document metadata is already configured in the route and the root route generally owns the HTML document.

### 2. Verify all editable profile configuration

The route reads values from `@/lib/portfolio.config`:

- `portfolioConfig.email`
- `portfolioConfig.profile.github`
- `portfolioConfig.profile.linkedin`
- `portfolioConfig.location`
- `portfolioConfig.degree`
- `portfolioConfig.university`
- `portfolioConfig.specialization`
- `portfolioConfig.resumePath`
- `portfolioConfig.socialProjects`
- `portfolioNavigation`
- `areasOfInterest`
- `skillGroups`

**Action:** Check that these values are accurate and editable in one place. Confirm email, LinkedIn, GitHub, resume path, project repository URLs, degree, university, specialization, and project descriptions before publishing. Do not replace missing values with invented details.

### 3. Check project repository links

**Observation:** Project cards show a real external link only when `project.repository` is present; otherwise they display “Repo link not shared”.

**Action:** Add only verified repository URLs. Confirm every external URL opens the intended repository and uses a valid HTTPS URL.

### 4. Verify the contact backend

**Observation:** The form performs client-side schema validation and invokes `submitPortfolioContact` using `useServerFn`. It displays a success message when the returned result has `success: true`.

**Not verified:** The implementation of `contactSubmissionSchema`, `submitPortfolioContact`, server-side validation, persistence/delivery, spam protection, rate limiting, and error handling were not included in the supplied source.

**Action:** Inspect `@/lib/contact-submissions` and `@/lib/contact-submissions.functions`. Submit a valid test message and confirm it reaches the intended destination or storage. Test invalid email, empty fields, overlong input, server failure, and repeated submissions. Keep the success message truthful to what the backend actually does.

### 5. Verify resume behavior

**Observation:** If `portfolioConfig.resumePath` is set, the page links to that path. Otherwise it links to an email request with the label “Resume available on request”.

**Action:** If enabling direct download, add the actual PDF to the public assets location expected by the framework and set a working path. Test the link in production. Do not set a path until the file exists.

### 6. Check asset availability

**Observation:** The route imports `@/assets/ai-orbit.jpg` and renders it in the hero section.

**Action:** Confirm the file exists at that path, loads in production, has suitable dimensions/compression, and has meaningful alt text. Confirm the `@` alias is configured in the TypeScript and bundler configuration.

### 7. Accessibility and interaction checks

The supplied route includes a skip link, labelled navigation, accessible mobile-menu state, form labels, field error descriptions, live form feedback, and icon labels.

Before release, test:
- Keyboard-only navigation and visible focus states.
- Mobile menu open/close and closing after a navigation selection.
- Reduced-motion behavior in the stylesheet.
- Contrast of muted text and small labels on dark backgrounds.
- Screen-reader announcements for validation and submission status.
- Whether the hero artwork and floating labels remain understandable at narrow widths.

### 8. SEO and social preview

The route declares a title, description, Open Graph title/description/type, and Twitter card type.

**Action:** Add a real deployed canonical URL and a suitable social preview image if the site framework supports them. Verify metadata in the rendered production HTML, not just in the source route.

### 9. Privacy and security

- Ensure contact submissions are handled securely and do not expose secrets or private API keys in client-side code.
- Add spam/rate limiting if the form is publicly accessible.
- Publish only contact details and profile links intended to be public.
- Avoid including private location coordinates. The supplied hero source contains coordinate text; verify whether those coordinates are intended as decorative copy and remove them if they imply a precise personal location.

## Release checklist

- [ ] Verify all values in `portfolio.config`.
- [ ] Replace missing social and repository links with verified URLs.
- [ ] Add and test the resume PDF, or retain the email-request fallback.
- [ ] Verify the contact server function actually delivers or stores submissions.
- [ ] Confirm `ai-orbit.jpg` exists and loads.
- [ ] Run the production build and resolve all TypeScript/build errors.
- [ ] Test desktop, tablet, and mobile layouts.
- [ ] Test every navigation link, external link, CTA, and form state.
- [ ] Check keyboard access, focus visibility, contrast, and reduced motion.
- [ ] Verify deployed page metadata and social preview.
- [ ] Remove or confirm the intent of the displayed coordinate text.

## Audit conclusion

The supplied route contains the structure for a polished five-section portfolio and includes meaningful accessibility, SEO, responsive-navigation, and form-handling patterns. However, this is a source-level review, not a verified production test. The configuration, styles, root document, imported components, image asset, server-side contact implementation, and deployed behavior must be checked before calling the site production-ready.
