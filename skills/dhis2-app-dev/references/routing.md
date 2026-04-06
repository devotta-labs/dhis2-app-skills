# Routing and Page Layout

DHIS2 apps run inside an iframe in the DHIS2 Global Shell, so they must use hash
routing (`createHashRouter`) to avoid conflicts with the platform's own routing.

## Recommended approach

We strongly recommend setting up a sidebar-based layout for navigation. Most DHIS2
apps grow into multi-page applications, and a sidebar gives users a consistent,
familiar way to move between sections — matching what they see in other DHIS2 apps.

Read `routing/sidebar.md` for the full implementation. It includes the sidebar
components, the page layout grid, a content width wrapper, route handles for
layout control (`collapseSidebar`, `fullWidth`), and the complete router wiring.
