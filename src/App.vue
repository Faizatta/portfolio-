<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, reactive, ref } from 'vue'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import Lenis from 'lenis'
import {
  ArrowDownRight, ArrowUpRight, Check, ExternalLink, Linkedin, Mail,
  MapPin, Menu, Phone, Send, X,
} from 'lucide-vue-next'
import { experience, projects, siteConfig, skills } from './data/portfolio'

gsap.registerPlugin(ScrollTrigger)

const navItems = [
  { id: 'home', label: 'Home' },
  { id: 'about', label: 'About' },
  { id: 'work', label: 'Work' },
  { id: 'contact', label: 'Contact' },
]

const expertise = [
  { number: '01', title: 'Frontend Development', detail: 'Responsive interfaces and polished experiences' },
  { number: '02', title: 'Backend Development', detail: 'APIs, authentication, and business logic' },
  { number: '03', title: 'Full-Stack Websites', detail: 'Complete products from interface to database' },
  { number: '04', title: 'ERP Systems', detail: 'Inventory, sales, purchases, and operations' },
  { number: '05', title: 'CRM Systems', detail: 'Customers, access control, and workflows' },
  { number: '06', title: 'Web Applications', detail: 'Practical software for real business needs' },
  { number: '07', title: 'Mobile Applications', detail: 'Cross-platform experiences with Flutter' },
]

const previewColors = {
  mint: 'linear-gradient(135deg, #c8d9d1, #718c89)',
  blue: 'linear-gradient(135deg, #c7d7e5, #7087a2)',
  orange: 'linear-gradient(135deg, #e2d2c2, #a6856b)',
  teal: 'linear-gradient(135deg, #c6dedb, #719e99)',
  violet: 'linear-gradient(135deg, #d5d0e4, #8e83a9)',
  sky: 'linear-gradient(135deg, #c6dbe5, #7899aa)',
}

const activeSection = ref('home')
const menuOpen = ref(false)
const projectFilter = ref('All')
const selectedProject = ref(null)
const form = reactive({ name: '', email: '', subject: '', message: '' })
const formErrors = reactive({ name: '', email: '', subject: '', message: '' })
const formStatus = ref('')

const projectFilters = ['All', 'Delivered work', 'Business systems', 'Product concepts', 'Academic work']
const filteredProjects = computed(() => projectFilter.value === 'All'
  ? projects
  : projects.filter((project) => project.category === projectFilter.value))
const hasEmail = computed(() => Boolean(siteConfig.email))
const hasPhone = computed(() => Boolean(siteConfig.phone))
const mailHref = computed(() => hasEmail.value ? `mailto:${siteConfig.email}` : undefined)
const phoneHref = computed(() => hasPhone.value ? `tel:${siteConfig.phone.replace(/\s/g, '')}` : undefined)

function scrollTo(id) {
  document.getElementById(id)?.scrollIntoView({ behavior: 'smooth' })
  menuOpen.value = false
}

function validateForm() {
  Object.keys(formErrors).forEach((key) => { formErrors[key] = '' })
  if (!form.name.trim()) formErrors.name = 'Please add your name.'
  if (!/^\S+@\S+\.\S+$/.test(form.email)) formErrors.email = 'Please add a valid email.'
  if (!form.subject.trim()) formErrors.subject = 'Please add a subject.'
  if (!form.message.trim()) formErrors.message = 'Please add a short message.'
  return !Object.values(formErrors).some(Boolean)
}

function submitForm() {
  formStatus.value = ''
  if (!validateForm()) return
  formStatus.value = siteConfig.contactFormEndpoint
    ? 'Delivery is configured for this form. Connect the endpoint handler to send this message.'
    : 'Your message is ready, but delivery is not configured yet. Add an endpoint in src/data/portfolio.js before publishing.'
}

let lenis
let tickerCallback
let sectionObserver
let animationContext

onMounted(async () => {
  await nextTick()
  const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches

  if (!reduceMotion) {
    lenis = new Lenis({ duration: 1.1, smoothWheel: true })
    lenis.on('scroll', ScrollTrigger.update)
    tickerCallback = (time) => lenis.raf(time * 1000)
    gsap.ticker.add(tickerCallback)
    gsap.ticker.lagSmoothing(0)
  }

  animationContext = gsap.context(() => {
    if (reduceMotion) return
    gsap.from('.hero-kicker, .hero-copy, .hero-meta', { opacity: 0, y: 22, duration: .8, stagger: .12, ease: 'power3.out', delay: .15 })
    gsap.from('.hero-name-line', { opacity: 0, yPercent: 100, duration: 1.05, stagger: .13, ease: 'power4.out', delay: .25 })
    gsap.from('.hero-mark', { opacity: 0, scale: .85, rotate: -5, duration: 1.1, ease: 'power3.out', delay: .35 })
    gsap.utils.toArray('.scroll-reveal').forEach((element) => {
      gsap.from(element, {
        opacity: 0, y: 45, duration: .9, ease: 'power3.out',
        scrollTrigger: { trigger: element, start: 'top 86%', once: true },
      })
    })
    gsap.to('.hero-mark', { y: 70, ease: 'none', scrollTrigger: { trigger: '#home', start: 'top top', end: 'bottom top', scrub: true } })
  })

  sectionObserver = new IntersectionObserver((entries) => {
    entries.forEach((entry) => { if (entry.isIntersecting) activeSection.value = entry.target.id })
  }, { rootMargin: '-24% 0px -64% 0px', threshold: 0 })
  document.querySelectorAll('main section[id]').forEach((section) => sectionObserver.observe(section))
})

onBeforeUnmount(() => {
  sectionObserver?.disconnect()
  animationContext?.revert()
  if (lenis) {
    lenis.destroy()
    gsap.ticker.remove(tickerCallback)
  }
})
</script>

<template>
  <div class="site-shell editorial-shell">
    <header class="site-nav">
      <nav class="container nav-inner" aria-label="Main navigation">
        <button class="brand-lockup" aria-label="Back to home" @click="scrollTo('home')"><span class="brand-mark">FA</span><span>Faiz Atta</span></button>
        <div class="availability"><span class="availability-dot"></span> Available for select work</div>
        <div class="desktop-nav"><button v-for="item in navItems" :key="item.id" class="nav-link" :class="{ active: activeSection === item.id }" @click="scrollTo(item.id)">{{ item.label }}</button><button class="nav-contact" @click="scrollTo('contact')">Get in touch <ArrowUpRight :size="15" /></button></div>
        <button class="menu-toggle" aria-label="Toggle navigation" :aria-expanded="menuOpen" @click="menuOpen = !menuOpen"><X v-if="menuOpen" :size="19" /><Menu v-else :size="19" /></button>
      </nav>
      <div v-if="menuOpen" class="mobile-nav"><button v-for="item in navItems" :key="item.id" class="nav-link" @click="scrollTo(item.id)">{{ item.label }}</button><button class="nav-contact" @click="scrollTo('contact')">Get in touch <ArrowUpRight :size="15" /></button></div>
    </header>

    <main>
      <section id="home" class="hero-section editorial-light">
        <div class="hero-lines" aria-hidden="true"><span></span><span></span><span></span></div>
        <div class="container hero-inner">
          <div class="hero-kicker"><span>01</span><span>Full-Stack Web Developer</span></div>
          <div class="hero-name" aria-label="Faiz Atta"><span class="hero-name-line">FAIZ</span><span class="hero-name-line hero-name-offset">ATTA</span></div>
          <div class="hero-mark" aria-hidden="true"><span>FA</span><small>Web<br />Developer</small></div>
          <div class="hero-copy"><p>I build digital tools that make complex business work feel simple.</p><button class="text-link dark-link" @click="scrollTo('work')">Explore selected work <ArrowDownRight :size="16" /></button></div>
          <div class="hero-meta"><span>Based in Rawalpindi, Pakistan</span><span>Laravel · Vue.js · ASP.NET Core</span></div>
          <div class="scroll-cue"><span>Scroll to explore</span><span class="scroll-cue-line"></span></div>
        </div>
      </section>

      <section id="about" class="about-section editorial-dark">
        <div class="container about-inner"><div class="section-index">02 / 05</div><div class="about-display"><span>/</span>ABOUT</div><div class="about-arrow"><ArrowDownRight :size="78" stroke-width="1" /></div><div class="about-copy scroll-reveal"><p class="eyebrow-light">A developer who thinks in systems</p><p class="about-lead">I turn real-world workflows into focused, dependable software.</p><p class="about-body">With experience across Laravel, Vue.js, REST APIs, Flutter, Python, Java, and ASP.NET Core, I work across the stack to build products that are useful, maintainable, and ready for the people who rely on them.</p><button class="text-link light-link" @click="scrollTo('skills')">See my expertise <ArrowDownRight :size="16" /></button></div></div>
      </section>

      <section id="skills" class="expertise-section editorial-light">
        <div class="container"><div class="section-index dark-index scroll-reveal">03 / 05</div><div class="section-heading-row scroll-reveal"><div><p class="eyebrow-dark">What I can do</p><h2 class="editorial-heading">I build the<br /><em>whole thing.</em></h2></div><p class="section-intro">From the first interface to the backend architecture, I create practical digital products for businesses, teams, and everyday users.</p></div><div class="expertise-list"> <article v-for="item in expertise" :key="item.number" class="expertise-row scroll-reveal"><span class="expertise-number">{{ item.number }}</span><h3>{{ item.title }}</h3><span class="expertise-detail">{{ item.detail }}</span><ArrowUpRight class="expertise-arrow" :size="20" /></article></div><div class="tool-strip scroll-reveal"><span>Also working with</span><span v-for="skill in skills" :key="skill.name">{{ skill.name }}</span></div></div>
      </section>

      <section id="work" class="work-section editorial-dark">
        <div class="container"><div class="section-index scroll-reveal">04 / 05</div><div class="section-heading-row work-heading scroll-reveal"><div><p class="eyebrow-light">Selected work</p><h2 class="editorial-heading light-heading">Thoughtful systems.<br /><em>Real outcomes.</em></h2></div><p class="section-intro light-intro">Delivered work, active concepts, and one confidential academic project—labeled clearly and presented with context.</p></div><div class="filter-row scroll-reveal" role="tablist" aria-label="Filter projects"><button v-for="filter in projectFilters" :key="filter" :class="{ selected: projectFilter === filter }" role="tab" :aria-selected="projectFilter === filter" @click="projectFilter = filter">{{ filter }}</button></div><div class="project-grid editorial-project-grid"><article v-for="project in filteredProjects" :key="project.title" class="editorial-project scroll-reveal"><div class="project-visual" :style="{ '--preview': previewColors[project.accent] }"><div class="visual-window"><div class="visual-top"><span></span><span></span><span></span><small>{{ project.label }}</small></div><div class="visual-content"><div class="visual-sidebar"><i></i><i></i><i></i><i></i></div><div class="visual-dashboard"><b></b><em></em><div class="visual-blocks"><i></i><i></i><i></i></div><div class="visual-bars"><i></i><i></i><i></i><i></i><i></i></div></div></div></div></div><div class="project-info"><div class="project-meta"><span :class="{ muted: project.status !== 'Delivered' }">{{ project.status }}</span><span>{{ project.category }}</span></div><h3>{{ project.title }}</h3><p>{{ project.description }}</p><div class="project-stack"><span v-for="tag in project.stack" :key="tag">{{ tag }}</span></div><button class="text-link light-link" @click="selectedProject = project">View details <ArrowUpRight :size="15" /></button></div></article></div></div>
      </section>

      <section id="experience" class="experience-section editorial-light"><div class="container"><div class="section-index dark-index scroll-reveal">Career /</div><div class="section-heading-row scroll-reveal"><div><p class="eyebrow-dark">Experience</p><h2 class="editorial-heading">Learning by<br /><em>building.</em></h2></div><p class="section-intro">A practical path through professional development, frontend craft, backend systems, and applied software studies.</p></div><div class="experience-list"><article v-for="item in experience" :key="item.company + item.date" class="experience-row scroll-reveal"><div><span>{{ item.date }}</span><small>{{ item.mode }}</small></div><div><h3>{{ item.role }}</h3><p class="experience-company">{{ item.company }}</p><p class="experience-description">{{ item.description }}</p></div></article></div></div></section>

      <section id="contact" class="contact-section editorial-light"><div class="contact-rule"></div><div class="container contact-inner"><div class="section-index dark-index">05 / 05</div><div class="contact-arrow"><ArrowDownRight :size="86" stroke-width="1" /></div><p class="eyebrow-dark scroll-reveal">Have a project in mind?</p><h2 class="contact-heading scroll-reveal">LET'S WORK<br /><em>TOGETHER.</em></h2><div class="contact-grid"><div class="contact-details scroll-reveal"><p>Have a problem worth solving? Send a message and let’s talk about what you’re building.</p><a v-if="hasEmail" :href="mailHref"><Mail :size="16" /> {{ siteConfig.email }}</a><a v-if="hasPhone" :href="phoneHref"><Phone :size="16" /> {{ siteConfig.phone }}</a><span><MapPin :size="16" /> {{ siteConfig.location }}</span><a class="linkedin-link" :href="siteConfig.linkedin" target="_blank" rel="noopener noreferrer"><Linkedin :size="16" /> LinkedIn <ExternalLink :size="13" /></a></div><form class="contact-form scroll-reveal" novalidate @submit.prevent="submitForm"><div class="form-row"><label>01 / Name<input v-model="form.name" class="editorial-input" :aria-invalid="Boolean(formErrors.name)" placeholder="Your name" /><small v-if="formErrors.name">{{ formErrors.name }}</small></label><label>02 / Email<input v-model="form.email" type="email" class="editorial-input" :aria-invalid="Boolean(formErrors.email)" placeholder="you@example.com" /><small v-if="formErrors.email">{{ formErrors.email }}</small></label></div><label>03 / Subject<input v-model="form.subject" class="editorial-input" :aria-invalid="Boolean(formErrors.subject)" placeholder="What would you like to build?" /><small v-if="formErrors.subject">{{ formErrors.subject }}</small></label><label>04 / Message<textarea v-model="form.message" class="editorial-input message-input" :aria-invalid="Boolean(formErrors.message)" placeholder="Tell me a little about the project..."></textarea><small v-if="formErrors.message">{{ formErrors.message }}</small></label><div class="form-submit"><button class="dark-button" type="submit">Send a message <Send :size="15" /></button><p v-if="formStatus" role="status">{{ formStatus }}</p></div></form></div><div class="footer-name">FAIZ ATTA</div></div></section>
    </main>

    <footer class="site-footer"><div class="container"><span>© {{ new Date().getFullYear() }} Faiz Atta</span></div></footer>

    <div v-if="selectedProject" class="project-modal" role="dialog" aria-modal="true" :aria-label="selectedProject.title" @click.self="selectedProject = null"><div class="modal-card"><div class="modal-head"><div><span class="modal-status">{{ selectedProject.status }}</span><h2>{{ selectedProject.title }}</h2></div><button class="modal-close" aria-label="Close project details" @click="selectedProject = null"><X :size="18" /></button></div><p>{{ selectedProject.description }}</p><div class="modal-stack"><span v-for="tag in selectedProject.stack" :key="tag">{{ tag }}</span></div><div class="modal-note"><Check :size="15" /> Project status is labeled from available evidence.</div></div></div>
  </div>
</template>
