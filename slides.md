---
layout: timeline
currentDate: '2020-11-05'
---
# In the beginning
There was nothing

---
layout: timeline
currentDate: '2021-11-05'
---

<CommitInfo 
  title="Initial project setup and repository creation (#1)"
  author="hmm"
  message="Set up the project repository with initial structure, configuration, and dependencies. Created the basic framework for the application."
  changes="7 files changed, 342 insertions(+), 0 deletions(-)"
  :files="['package.json', 'README.md', 'tsconfig.json', 'src/index.ts', 'src/config.ts', '.gitignore', '.eslintrc']"
  date="2023-01-15"
/>

---
layout: timeline
currentDate: '2022-02-09'
---

<CommitInfo 
  title=":tada: Initial commit newest version of code (#9)"
  author="Aleksander Mats Karlsson"
  message="Source code written by @kuleb <kuleb@equinor.com>"
  changes="18 files changed, 18378 insertions(+)"
  :files="['src/banner.py','src/eclread.py','src/eos.py','src/fluid_magic.py','src/fm_blackoil.py','src/fm_boc_converter.py','src/fm_calculator.py','src/fm_converters.py','src/fm_e300_data.py','src/fm_file_handle.py','src/fm_fluid_gradients.py','src/fm_gamma_plus.py','src/fm_multi_phase_meters.py','src/fm_numerics.py','src/fm_postecl.py','src/fm_process.py','src/fm_process_converters.py','src/pvt.py']"
  date="2022-02-09"
/>

---
layout: timeline
---

# Something

- MVP functionality completed
- First user testing session
- Feedback integration plan

---
layout: timeline
currentDate: '2023-05-20'
---

# Major Challenge
May 20, 2023
- Performance optimization needed
- Scaling architecture
- Team retrospective 

---
layout: timeline
currentDate: '2023-07-15'
---

# Mid-Project Review
July 15, 2023

- Stakeholder demo
- Feature prioritization
- Growth metrics analysis

---
layout: timeline
currentDate: '2023-10-05'
---

# Final Testing
October 5, 2023

- Comprehensive QA
- User acceptance testing
- Documentation review

---
layout: timeline
currentDate: '2023-11-30'
---

# Project Delivery
November 30, 2023

- Production deployment
- Customer onboarding
- Knowledge transfer complete

---
layout: timeline
currentDate: '2023-12-10'
---

# oioio

<CommitInfo 
  title="Move tests out of numeric file to separate file (#62)"
  author="Arvid Gräns"
  message="Moving tests out of fm_numeric.py into a separate file in test folder. This improves structure and testability."
  changes="3 files changed, 61 insertions(+), 43 deletions(-)"
  :files="['requirements.txt', 'src/fm_numerics.py', 'tests/numerics_test.py']"
  date="2023-11-28"
/>

---
layout: timeline
currentDate: '2023-12-15'
---

# Critical Security Fix

<CommitInfo 
  title="Fix security vulnerability in authentication module (#78)"
  author="Magnus Nordin"
  message="Fixed a critical security vulnerability in the authentication module that could allow unauthorized access. Updated dependency versions and added additional validation steps."
  changes="5 files changed, 87 insertions(+), 32 deletions(-)"
  :files="['package.json', 'src/auth/authenticate.js', 'src/auth/validation.js', 'tests/auth_test.js', 'README.md']"
  date="2023-12-03"
/>

---
layout: timeline
---

# Security Impact Analysis
## Associated with the Critical Security Fix

- No user data was compromised
- Attack vector required authenticated access
- Deployed to production within 24 hours of discovery
- All dependencies updated to latest secure versions