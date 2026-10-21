# Debugging and problem-solving case studies

These case studies are based on problems reported and fixes attempted during development. They are **not** claims that each fix was verified by automated end-to-end tests.

## 1. Marketplace fails with `paid is not defined`

**Symptom:** The UI displayed `Could not load marketplace: paid is not defined`.

**Diagnosis:** A payment/order-tracking variable was referenced outside the scope where it had been declared, in unrelated admin logic. This caused a JavaScript `ReferenceError` during marketplace initialization.

**Fix applied:** Remove the misplaced reference and keep order-specific logic inside the order rendering flow.

**Lesson:** Large single-file applications make it easy for an unrelated edit to affect initialization. Use linting, modular functions, and a regression smoke test for authentication, listings, and admin screens.

**Verification still needed:** Open the site with a normal account and an admin account; confirm the marketplace loads without console errors.

## 2. Login stopped working after an admin report fix

**Symptom:** Sign-in UI stopped behaving correctly after a dashboard change.

**Diagnosis:** Initialization still tried to remove an old DOM element that had already been removed from the page. A null-element operation stopped subsequent JavaScript setup.

**Fix applied:** Guard or remove the obsolete DOM manipulation and verify event-handler setup continues.

**Lesson:** When removing UI components, search for all references to their IDs; run a startup smoke test, not just a syntax check.

**Verification still needed:** Test signed-out → login → account → sign-out → login again.

## 3. Admin reports were not visible

**Symptom:** The admin report queue appeared empty even though the user expected reports.

**Diagnosis:** Duplicate report-list IDs allowed the JavaScript selector to target a hidden legacy section rather than the visible Admin Center.

**Fix applied:** Remove the old duplicate section and keep a single report-list container.

**Lesson:** IDs must be unique. Add a DOM validation test to detect duplicates before deployment.

**Verification still needed:** Insert a test report and confirm it appears for a moderator/owner, not for ordinary users.

## 4. Sign-out dropdown was clipped or unclickable

**Symptom:** Hovering over My Account showed a Sign Out item that could be cut off or disappear when moving the cursor.

**Diagnosis:** CSS overflow, stacking context, and a hover gap interfered with the dropdown.

**Fix applied:** Adjust dropdown positioning, stacking, overflow and hover continuity.

**Lesson:** Interactive UI must be tested with pointer and keyboard, on every page layout and at narrow viewport sizes.

**Verification still needed:** Desktop hover, keyboard focus, mobile tap and sign-out on each page.

## 5. Mobile content and navigation clipping

**Symptom:** Header content, statistics cards and login text were clipped at certain viewport sizes.

**Diagnosis:** Fixed widths, header overlap and responsive spacing rules did not consistently fit smaller viewports.

**Fix applied:** Scoped responsive CSS and adjusted sizing/overflow without intentionally altering desktop styles.

**Lesson:** Test at multiple viewport widths and with long labels. A syntax-valid page can still be visually broken.

**Verification still needed:** 320, 375, 390, 768 and desktop widths, with signed-in and signed-out states.

## Suggested portfolio talking point

> I built a student marketplace prototype and iteratively debugged frontend and integration problems, including a JavaScript scope error, duplicate DOM IDs, startup failures, and responsive navigation issues. I documented the root causes and added a repeatable regression checklist to reduce future breakage.

This accurately describes the development process without claiming unverified production security or automated test coverage.
