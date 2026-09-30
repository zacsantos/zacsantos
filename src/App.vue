<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, ref } from 'vue'
import certificateUrl from '../assets/CertificateOfAppreciation.jpg'
import {
  ArrowDown, ArrowLeft, ArrowRight, ArrowUpRight, Award, Braces, Car, Check, ChevronDown,
  Clock, Database, FileCheck2, FileText, GitBranch, Menu, MessageSquare, Play, Radio,
  RotateCcw, Send, Sparkles, Terminal, X,
} from '@lucide/vue'

const menuOpen = ref(false)
const activeExperience = ref(0)
const activeProject = ref(null)
const certificateOpen = ref(false)

const experiences = [
  {
    period: 'RECENT WORK',
    role: 'Internal Platforms & Automation',
    company: 'Enterprise operations · identity protected',
    summary: 'Full-stack tools that replace fragmented email work with visible, dependable operational flows.',
    projects: [
      {
        number: '01', title: 'Shared contract room', type: 'Collaboration · Version control',
        intro: 'One protected workspace for requests, revisions, files, decisions and supplier conversation.',
        accent: '#2f6fed', preview: 'contract',
        challenge: 'Contract revisions moved through long email threads. Different participants could act on different files, while decisions and context became difficult to recover.',
        response: 'I built a guided workspace around a single current version, visible status history, structured attachments and a contextual chat. Internal participants and external collaborators see the same request without exposing the wider environment.',
        contribution: ['Full-stack development', 'Workflow architecture', 'Version history', 'Collaboration UX'],
      },
      {
        number: '02', title: 'Structured invoice request', type: 'Operational forms · Review flow',
        intro: 'A role-aware request that turns emailed forms into a visible sequence of accountable steps.',
        accent: '#72a8e8', preview: 'invoice',
        challenge: 'People exchanged PDF forms and corrections through email, making ownership, completeness and the latest status difficult to understand.',
        response: 'I transformed the process into a structured digital request with section-level responsibility, validation, document previews, review states and a shared history from creation to completion.',
        contribution: ['Process modelling', 'Vue interface', 'Validation rules', 'Status architecture'],
      },
      {
        number: '03', title: 'Reviewed data handoff', type: 'Data reuse · Assisted submission',
        intro: 'Reuse data that already exists, verify it once, then submit it without retyping.',
        accent: '#4a9fd8', preview: 'handoff',
        challenge: 'Employees repeatedly re-entered information that already existed in an internal data source before sending it to an external public service.',
        response: 'I created a middle layer that gathers the existing record, maps it into a reviewable form, flags missing fields and keeps a human confirmation before the simulated final submission.',
        contribution: ['Data mapping', 'API orchestration', 'Review UX', 'Error handling'],
      },
      {
        number: '04', title: 'Automation operations hub', type: 'Python orchestration · FIFO queue',
        intro: 'A front door for requesting, observing and managing file-based automations at scale.',
        accent: '#657bd7', preview: 'automation', featured: true,
        challenge: 'Useful Python automations lived behind technical execution steps and operated on shared files without one place to manage demand, progress or results.',
        response: 'I designed and built a reusable orchestration layer: users choose an automation, supply permitted inputs, enter a FIFO queue and follow status, bounded live logs and generated results from one request page.',
        contribution: ['Platform architecture', 'Python execution', 'Queue management', 'Live operational UX'],
      },
    ],
  },
  {
    period: 'EARLIER WORK',
    role: 'Connected Systems & Data',
    company: 'Mobility technology · identity protected',
    summary: 'Data-focused work exploring how connected systems can transmit less, predict better and operate reliably.',
    projects: [
      {
        number: '05', title: 'Adaptive telemetry profiles', type: 'Connected data · Optimization',
        intro: 'Tailoring what a connected product transmits to reduce unnecessary data while preserving useful service.',
        accent: '#367fd3', preview: 'vehicle',
        challenge: 'A uniform transmission strategy can send more connected-product data than each service or customer context actually needs.',
        response: 'The project explored profiles that select the most relevant signals for each context, reducing redundant payloads while keeping the experience useful and measurable.',
        contribution: ['Data analysis', 'Python', 'Cloud data', 'Optimization logic'],
      },
      {
        number: '06', title: 'Connectivity-aware delivery', type: 'Prediction · Reliable transfer',
        intro: 'Anticipating connectivity windows so data packets can move at a better moment.',
        accent: '#87b9ed', preview: 'network',
        challenge: 'Mobile connected systems can move in and out of coverage, creating avoidable retries, delayed packets and inefficient transmission.',
        response: 'The concept uses predicted connectivity conditions to help schedule transfer before an interruption or defer it until a more reliable window.',
        contribution: ['Predictive analysis', 'Data structures', 'Transmission strategy', 'Technical prototyping'],
      },
      {
        number: '07', title: 'Regional safety follow-up bot', type: 'Functional safety · Workflow automation',
        intro: 'Turning global pending-work data into locally owned, actionable follow-ups without manual sorting.',
        accent: '#5571c9', preview: 'safetybot', recognition: '1st place · Time optimization category · 2024',
        challenge: 'A global work tracker reflected one ownership structure, while the regional team used different feature owners. Someone had to repeatedly reconcile pending items against a spreadsheet and contact the correct people.',
        response: 'I helped build a Python and Pandas workflow that collected fictional pending items through a REST-style interface, matched each feature against a regional ownership matrix, grouped the results by owner and prepared targeted collaboration messages. The portfolio simulation keeps the mapping logic while removing every real name and service.',
        contribution: ['Python & Pandas', 'REST / JSON integration', 'Spreadsheet validation', 'International rollout'],
      },
    ],
  },
]

const automations = [
  {
    id: 'organize', name: 'Archive organizer', description: 'Classify and rename fictional documents.', input: '18 sample files',
    output: '18 files organized · 0 conflicts',
    logs: ['Validating 18 sample documents…', 'Creating safe workspace /mock/run-1042', 'Classifying filenames and dates…', 'Writing manifest.json', 'Completed without conflicts.'],
  },
  {
    id: 'reconcile', name: 'Table reconciler', description: 'Compare two invented workbooks.', input: '2 mock tables',
    output: '42 rows matched · 3 flagged for review',
    logs: ['Reading fictional source tables…', 'Normalizing headers and identifiers…', 'Comparing 45 mock rows…', 'Flagging 3 differences for review', 'Reconciliation report is ready.'],
  },
  {
    id: 'summary', name: 'Summary builder', description: 'Turn sample activity into a weekly brief.', input: '7 days of fake activity',
    output: '1 summary · 4 charts · 6 highlights',
    logs: ['Loading fictional activity records…', 'Grouping events by category…', 'Calculating sample indicators…', 'Rendering four lightweight charts', 'Weekly brief generated successfully.'],
  },
]

const selectedAutomation = ref(automations[0])
const automationStatus = ref('idle')
const automationLogs = ref([])
const automationProgress = ref(0)
const automationOutput = ref('')
let automationTimers = []
let experienceScrollTimer = null

const orderedProjectList = computed(() => experiences.flatMap((experience) => orderedProjects(experience)))
const projectIndex = computed(() => activeProject.value ? orderedProjectList.value.findIndex((item) => item.title === activeProject.value.title) : -1)

function toggleExperience(index, event) {
  const previousIndex = activeExperience.value
  const opening = previousIndex !== index
  activeExperience.value = opening ? index : -1
  if (!opening) return

  const experienceItem = event.currentTarget.closest('.experience-item')
  nextTick(() => {
    window.requestAnimationFrame(() => {
      const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
      const collapsingSectionAbove = previousIndex >= 0 && previousIndex < index
      const layoutDelay = !reduceMotion && collapsingSectionAbove ? 520 : 0

      if (experienceScrollTimer) window.clearTimeout(experienceScrollTimer)
      experienceScrollTimer = window.setTimeout(() => {
        experienceItem?.scrollIntoView({ behavior: reduceMotion ? 'auto' : 'smooth', block: 'start' })
        experienceScrollTimer = null
      }, layoutDelay)
    })
  })
}
function orderedProjects(experience) { return [...experience.projects].sort((a, b) => Number(Boolean(b.featured)) - Number(Boolean(a.featured))) }
function shouldIsolateFeatured(project, experience) { return Boolean(project.featured) && (experience.projects.length - 1) % 2 === 0 }
function clearAutomationTimers() { automationTimers.forEach((timer) => window.clearTimeout(timer)); automationTimers = [] }
function resetAutomation() { clearAutomationTimers(); automationStatus.value = 'idle'; automationLogs.value = []; automationProgress.value = 0; automationOutput.value = '' }
function selectAutomation(automation) {
  if (automationStatus.value === 'running' || automationStatus.value === 'queued') return
  selectedAutomation.value = automation
  resetAutomation()
}
function runMockAutomation() {
  resetAutomation()
  automationStatus.value = 'queued'
  automationLogs.value = ['Request accepted as MOCK-1042', 'Queue position: 1']
  automationProgress.value = 8
  automationTimers.push(window.setTimeout(() => { automationStatus.value = 'running'; automationLogs.value.push('Worker available — starting isolated preview'); automationProgress.value = 18 }, 650))
  selectedAutomation.value.logs.forEach((line, index) => {
    automationTimers.push(window.setTimeout(() => { automationLogs.value.push(line); automationProgress.value = Math.min(94, 30 + (index + 1) * 13) }, 1250 + index * 600))
  })
  automationTimers.push(window.setTimeout(() => { automationStatus.value = 'complete'; automationProgress.value = 100; automationOutput.value = selectedAutomation.value.output }, 1250 + selectedAutomation.value.logs.length * 600 + 350))
}
function openProject(project) { activeProject.value = project; resetAutomation(); document.body.classList.add('modal-open') }
function closeProject() { clearAutomationTimers(); certificateOpen.value = false; activeProject.value = null; document.body.classList.remove('modal-open') }
function moveProject(direction) {
  const all = orderedProjectList.value
  activeProject.value = all[(projectIndex.value + direction + all.length) % all.length]
  resetAutomation()
}
function onKeydown(event) {
  if (event.key === 'Escape' && certificateOpen.value) { certificateOpen.value = false; return }
  if (event.key === 'Escape') closeProject()
  if (activeProject.value?.preview !== 'automation' && event.key === 'ArrowRight') moveProject(1)
  if (activeProject.value?.preview !== 'automation' && event.key === 'ArrowLeft') moveProject(-1)
}
onMounted(() => window.addEventListener('keydown', onKeydown))
onBeforeUnmount(() => {
  window.removeEventListener('keydown', onKeydown)
  clearAutomationTimers()
  if (experienceScrollTimer) window.clearTimeout(experienceScrollTimer)
})
</script>

<template>
  <div class="site-shell">
    <header class="site-header">
      <a class="wordmark" href="#top" aria-label="Isaac Luís Silva Santos, home">
        <span class="wordmark-part" aria-hidden="true">
          <strong class="wordmark-initial">I</strong><span class="wordmark-tail given-name">saac Luís Silva&nbsp;</span>
        </span>
        <span class="wordmark-part surname" aria-hidden="true">
          <strong class="wordmark-initial">S</strong><span class="wordmark-tail surname-tail">antos</span>
        </span>
        <i aria-hidden="true">.</i>
      </a>
      <button class="menu-button" aria-label="Toggle navigation" @click="menuOpen = !menuOpen"><X v-if="menuOpen" :size="22" /><Menu v-else :size="22" /></button>
      <nav :class="['nav-links', { open: menuOpen }]" aria-label="Main navigation">
        <a href="#about" @click="menuOpen = false">About</a><a href="#experience" @click="menuOpen = false">Experience</a>
        <a href="https://www.linkedin.com/in/zacsantos" target="_blank" rel="noreferrer">LinkedIn <ArrowUpRight :size="15" /></a>
      </nav>
    </header>

    <main id="top">
      <section class="hero">
        <div class="hero-kicker"><span></span> Software · data · automation</div>
        <h1>I turn busy work into <em>systems.</em></h1>
        <div class="hero-bottom">
          <p>I’m Isaac Santos, a developer and data professional building internal platforms that connect people, information and automation—without losing the human decision in the middle.</p>
          <a class="round-link" href="#experience" aria-label="Go to experience"><ArrowDown :size="24" /></a>
        </div>
        <div class="hero-stamp" aria-hidden="true"><Sparkles :size="18" /><span>BUILD THE<br />BETTER PATH</span></div>
      </section>

      <section id="about" class="about section-grid">
        <div class="section-label">01 / About</div>
        <div class="about-copy">
          <p class="large-copy">I build bridges between <span>business workflows, reliable data and practical automation.</span></p>
          <div class="about-columns">
            <p>My projects often begin where email chains, repeated forms, shared files and manual re-entry make everyday work slower or less reliable. I map the real process, then shape a simpler system around it.</p>
            <p>Every case below is intentionally reconstructed. Names, records, screens, integrations and outputs are fictional; the product problem, architecture pattern and my contribution are the parts that remain true.</p>
          </div>
        </div>
      </section>

      <section id="experience" class="experience-section">
        <div class="section-grid section-heading"><div class="section-label">02 / Experience</div><div><h2>Selected systems</h2><p>Open an experience, then choose a reconstructed project preview.</p></div></div>
        <div class="experience-list">
          <article v-for="(experience, index) in experiences" :key="experience.role" class="experience-item">
            <button class="experience-trigger" :aria-expanded="activeExperience === index" @click="toggleExperience(index, $event)">
              <span class="experience-period">{{ experience.period }}</span>
              <span class="experience-main"><strong>{{ experience.role }}</strong><small>{{ experience.company }}</small></span>
              <span class="experience-summary">{{ experience.summary }}</span>
              <span :class="['chevron', { active: activeExperience === index }]"><ChevronDown :size="22" /></span>
            </button>
            <div :class="['project-drawer', { open: activeExperience === index }]">
              <div class="drawer-inner">
                <button v-for="project in orderedProjects(experience)" :key="project.title" :class="['project-card', { featured: project.featured, isolated: shouldIsolateFeatured(project, experience) }]" @click="openProject(project)">
                  <div class="project-visual" :style="{ '--accent': project.accent }">
                    <div v-if="project.preview === 'contract'" class="mini-contract"><div class="contract-sheet"><FileText :size="20" /><span>v3 · current</span></div><div class="version-stack"><i></i><i></i><GitBranch :size="18" /></div><div class="chat-dot"><MessageSquare :size="17" /><b>3</b></div></div>
                    <div v-else-if="project.preview === 'invoice'" class="mini-invoice"><div class="mini-steps"><i class="done"></i><i class="active"></i><i></i></div><span></span><span></span><span class="short"></span><div><FileCheck2 :size="18" /> Ready for review</div></div>
                    <div v-else-if="project.preview === 'handoff'" class="mini-handoff"><div><Database :size="20" /><small>Source</small></div><span><i></i><i></i><i></i></span><div class="review"><Check :size="20" /><small>Review</small></div><span><i></i><i></i><i></i></span><div><Send :size="20" /><small>Send</small></div></div>
                    <div v-else-if="project.preview === 'automation'" class="mini-automation"><div class="auto-list"><span><Play :size="12" /> Archive</span><span><Play :size="12" /> Reconcile</span><span><Play :size="12" /> Summary</span></div><div class="auto-console"><b>FIFO / 01</b><i></i><i></i><i></i><small>ready</small></div></div>
                    <div v-else-if="project.preview === 'safetybot'" class="mini-safety-bot"><div><Database :size="18" /><small>Pending work</small></div><span>+</span><div><FileText :size="18" /><small>Owner matrix</small></div><span>→</span><div class="bot-message"><Send :size="18" /><small>3 grouped notes</small></div><b><Award :size="15" /> 1st</b></div>
                    <div v-else-if="project.preview === 'vehicle'" class="mini-vehicle"><Car :size="54" /><div><span v-for="n in 5" :key="n" :class="{ active: n < 4 }"></span></div><small>adaptive signal set</small></div>
                    <div v-else class="mini-network"><Radio :size="38" /><div class="packet-stream"><i v-for="n in 5" :key="n"></i></div><b>87%</b></div>
                    <span class="fictional-tag">RECONSTRUCTED</span>
                  </div>
                  <div class="project-meta"><span>{{ project.number }} / {{ project.type }}</span><ArrowUpRight :size="18" /></div><h3>{{ project.title }}</h3><p>{{ project.intro }}</p>
                </button>
              </div>
            </div>
          </article>
        </div>
      </section>

      <section class="principles section-grid"><div class="section-label">03 / Approach</div><div class="principle-list">
        <div><span>01</span><h3>See the whole flow</h3><p>Understand the people, files, handoffs, constraints and exceptions before choosing the interface.</p></div>
        <div><span>02</span><h3>Automate with visibility</h3><p>Make queues, progress, failures and human decisions clear instead of hiding them behind a button.</p></div>
        <div><span>03</span><h3>Reuse what already exists</h3><p>Move reliable information between systems without asking people to type the same thing twice.</p></div>
      </div></section>
      <section class="contact"><div class="contact-mark">Let’s build the<br /><em>better path.</em></div><a href="https://www.linkedin.com/in/zacsantos" target="_blank" rel="noreferrer">Start a conversation <ArrowUpRight :size="20" /></a></section>
    </main>

    <footer><span>© {{ new Date().getFullYear() }} Isaac Santos</span><span>Reconstructed with privacy in mind.</span><a href="https://www.linkedin.com/in/zacsantos" target="_blank" rel="noreferrer"><ArrowUpRight :size="17" /> LinkedIn</a></footer>

    <Teleport to="body">
      <div v-if="activeProject" class="modal" role="dialog" aria-modal="true" :aria-label="activeProject.title">
        <button class="modal-backdrop" aria-label="Close project" @click="closeProject"></button>
        <article :class="['case-study', { 'automation-case': activeProject.preview === 'automation' }]">
          <header class="case-header"><div><span>Reconstructed case · {{ activeProject.number }}</span><h2>{{ activeProject.title }}</h2></div><button class="close-button" aria-label="Close project" @click="closeProject"><X /></button></header>

          <div v-if="activeProject.preview === 'automation'" class="automation-lab" :style="{ '--accent': activeProject.accent }">
            <aside class="automation-picker">
              <span class="lab-label">Choose a fictional automation</span>
              <button v-for="automation in automations" :key="automation.id" :class="{ active: selectedAutomation.id === automation.id }" :disabled="automationStatus === 'running' || automationStatus === 'queued'" @click="selectAutomation(automation)">
                <span><Braces :size="17" />{{ automation.name }}</span><small>{{ automation.description }}</small>
              </button>
              <div class="input-summary"><FileText :size="17" /><span>Mock input</span><b>{{ selectedAutomation.input }}</b></div>
            </aside>
            <section class="automation-runner">
              <div class="runner-topline"><span class="lab-label">RUN / MOCK-1042</span><span :class="['run-status', automationStatus]"><i></i>{{ automationStatus }}</span></div>
              <div class="queue-strip"><div><Clock :size="17" /><span>FIFO QUEUE</span><b>{{ automationStatus === 'idle' ? 'Waiting to submit' : automationStatus === 'queued' ? 'Position 1' : automationStatus === 'complete' ? 'Finished' : 'Worker active' }}</b></div><div class="progress-track"><i :style="{ width: `${automationProgress}%` }"></i></div><strong>{{ automationProgress }}%</strong></div>
              <div class="mock-terminal" aria-live="polite"><div class="terminal-bar"><span><i></i><i></i><i></i></span><b><Terminal :size="14" /> fictional output</b></div><div class="terminal-lines"><p v-if="automationLogs.length === 0" class="muted">Select an automation and run the preview. Nothing will leave this page.</p><p v-for="(line, index) in automationLogs" :key="`${line}-${index}`"><span>{{ String(index + 1).padStart(2, '0') }}</span>{{ line }}</p><p v-if="automationStatus === 'running'" class="cursor-line">processing<span>_</span></p></div></div>
              <div v-if="automationOutput" class="mock-result"><Check :size="19" /><span>Mock result</span><b>{{ automationOutput }}</b></div>
              <div class="runner-actions"><button class="run-button" :disabled="automationStatus === 'running' || automationStatus === 'queued'" @click="runMockAutomation"><Play :size="17" /> {{ automationStatus === 'complete' ? 'Run again' : 'Run simulation' }}</button><button class="reset-button" :disabled="automationStatus === 'idle'" @click="resetAutomation"><RotateCcw :size="16" /> Reset</button></div>
            </section>
            <div class="privacy-note"><span></span> Local animation only · No Python, files, API or network calls</div>
          </div>

          <div v-else class="case-hero" :style="{ '--accent': activeProject.accent }">
            <div class="case-simulation">
              <div v-if="activeProject.preview === 'contract'" class="case-contract"><div class="case-document"><FileText :size="28" /><span>AGREEMENT_03</span><b>Current working version</b><small>Updated moments ago · fictional</small></div><div class="case-history"><span><GitBranch :size="18" /> Version history</span><i class="current">v3</i><i>v2</i><i>v1</i></div><div class="case-chat"><span><MessageSquare :size="18" /> Context</span><p>Revision ready for review.</p><p class="reply">I’ll check section 4.</p></div></div>
              <div v-else-if="activeProject.preview === 'invoice'" class="case-form-flow"><div class="form-progress"><span class="complete"><Check :size="16" /> Request</span><i></i><span class="complete"><Check :size="16" /> Review</span><i></i><span>Document</span></div><div class="form-panel"><small>FICTIONAL REQUEST / 024</small><b>Invoice support</b><span></span><span></span><span class="short"></span><button>Continue review</button></div></div>
              <div v-else-if="activeProject.preview === 'handoff'" class="case-handoff"><div class="sim-node"><Database :size="24" /><span>Internal source</span><b>12 mock fields found</b></div><div class="sim-connector"><i></i><span>FETCHING FICTIONAL DATA</span></div><div class="sim-node"><FileCheck2 :size="24" /><span>Human review</span><b>12 of 12 verified</b></div><div class="sim-connector"><i></i><span>READY</span></div><div class="sim-node success"><Send :size="24" /><span>External service</span><b>Submit preview</b></div></div>
              <div v-else-if="activeProject.preview === 'safetybot'" class="case-safety-bot"><div class="safety-flow"><div><Database :size="25" /><span>Global tracker</span><b>8 fictional pending items</b></div><i>+</i><div><FileText :size="25" /><span>Regional matrix</span><b>Feature → local owner</b></div><i>→</i><div><Braces :size="25" /><span>Python grouping</span><b>3 owner summaries</b></div><i>→</i><div><Send :size="25" /><span>Collaboration channel</span><b>Messages prepared</b></div></div><div class="award-card"><button class="certificate-trigger" aria-label="View the original certificate of appreciation" @click="certificateOpen = true"><Award :size="35" /><span>View certificate</span></button><small>2024 RECOGNITION</small><strong>1st place</strong><span>Time optimization category</span><p>The original certificate is available by choice and includes organization and teammate details.</p></div></div>
              <div v-else-if="activeProject.preview === 'vehicle'" class="case-vehicle"><Car :size="78" /><div class="signal-profile"><span v-for="(label, index) in ['Safety', 'Usage', 'Health', 'Comfort', 'Debug']" :key="label"><i :style="{ width: `${95 - index * 15}%` }"></i><b>{{ label }}</b></span></div><div class="payload-result"><small>MOCK PAYLOAD</small><strong>−31%</strong><span>redundant signals</span></div></div>
              <div v-else class="case-network"><div class="coverage-chart"><span v-for="n in 14" :key="n" :style="{ height: `${26 + ((n * 17) % 70)}%` }"></span><i></i></div><div class="network-event"><Radio :size="27" /><span>Predicted interruption</span><b>in 04:20</b></div><div class="packet-decision"><Check :size="25" /><span>Sample packet</span><b>Schedule before gap</b></div></div>
            </div>
            <div class="privacy-note"><span></span> Fictional interface · No company data or live connections</div>
          </div>

          <div class="case-content"><section><span>THE CHALLENGE</span><p>{{ activeProject.challenge }}</p></section><section><span>THE RESPONSE</span><p>{{ activeProject.response }}</p></section><section v-if="activeProject.recognition" class="recognition-summary"><Award :size="22" /><span>RECOGNITION</span><p>{{ activeProject.recognition }}</p></section><section class="contribution"><span>MY CONTRIBUTION</span><ul><li v-for="item in activeProject.contribution" :key="item">{{ item }}</li></ul></section></div>
          <footer class="case-footer"><button @click="moveProject(-1)"><ArrowLeft :size="18" /> Previous</button><span>{{ projectIndex + 1 }} / {{ orderedProjectList.length }}</span><button @click="moveProject(1)">Next <ArrowRight :size="18" /></button></footer>
        </article>
        <div v-if="certificateOpen" class="certificate-lightbox" role="dialog" aria-modal="true" aria-label="Original certificate of appreciation">
          <button class="certificate-backdrop" aria-label="Close certificate" @click="certificateOpen = false"></button>
          <figure>
            <button class="certificate-close" aria-label="Close certificate" @click="certificateOpen = false"><X :size="22" /></button>
            <img :src="certificateUrl" alt="Original 2024 certificate of appreciation for first place in the Time Optimization category" />
            <figcaption>Original certificate · Click outside or press Escape to close</figcaption>
          </figure>
        </div>
      </div>
    </Teleport>
  </div>
</template>
