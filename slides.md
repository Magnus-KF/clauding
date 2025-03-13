---
layout: timeline
currentDate: '2023-01-15'
---
<CommitInfo 
  title="Fix security vulnerability in authentication module (#78)"
  author="Magnus Nordin"
  message="Fixed a critical security vulnerability in the authentication module that could allow unauthorized access. Updated dependency versions and added additional validation steps."
  changes="5 files changed, 87 insertions(+), 32 deletions(-)"
  :files="['package.json', 'src/auth/authenticate.js', 'src/auth/validation.js', 'tests/auth_test.js', 'README.md']"
/>

---
layout: timeline
currentDate: '2023-03-10'
---

# First Milestone
March 10, 2023

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

# Key Project Commits

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