<script lang="ts">
	import { browser } from '$app/environment';

	// ─── Placeholder data (replace via JSON editor or AI prompt) ─────────────────
	const defaultCv = {
		personal: {
			fullName: 'Alex Johnson',
			title: 'Senior Frontend Engineer',
			email: 'alex.johnson@email.com',
			phone: '+1 (415) 555-0192',
			location: 'San Francisco, CA',
			summary:
				'Frontend engineer with 5+ years shipping production web apps at scale. Deep expertise in React, TypeScript, and modern build tooling. Passionate about performance, accessibility, and clean component architecture. Proven track record leading cross-functional delivery on B2B SaaS and consumer products.',
			links: [
				{ label: 'GitHub', url: 'github.com/alexjohnson' },
				{ label: 'LinkedIn', url: 'linkedin.com/in/alexjohnson' }
			]
		},
		sections: [
			{
				id: 'skills',
				title: 'Technical Skills',
				type: 'skills',
				items: [
					{ category: 'Frontend', values: ['React', 'Next.js', 'TypeScript', 'SvelteKit', 'TailwindCSS', 'GraphQL'] },
					{ category: 'Backend', values: ['Node.js', 'PostgreSQL', 'Redis', 'REST APIs', 'Prisma', 'Docker'] },
					{ category: 'Tooling', values: ['Vite', 'Webpack', 'Git', 'GitHub Actions', 'AWS', 'Figma'] }
				]
			},
			{
				id: 'experience',
				title: 'Work Experience',
				type: 'experience',
				items: [
					{
						title: 'Senior Frontend Engineer',
						subtitle: 'Stripe',
						date: 'Jan 2022 – Present',
						location: 'San Francisco, CA',
						description: '',
						details: [
							'Led a 4-engineer team to rebuild the merchant dashboard in Next.js, reducing LCP by 52% and cutting customer support tickets by 30%',
							'Architected a reusable component library (60+ components) adopted across 8 product teams, standardising design system tokens',
							'Integrated real-time data pipelines using WebSockets and React Query, enabling live transaction monitoring for 200k+ merchants'
						]
					},
					{
						title: 'Frontend Engineer',
						subtitle: 'Notion',
						date: 'Mar 2020 – Dec 2021',
						location: 'Remote',
						description: '',
						details: [
							'Built and shipped the collaborative editing toolbar, now used by 30M+ users, with conflict-free real-time sync',
							'Migrated legacy class components to React Hooks and Context API, reducing bundle size by 18%',
							'Owned end-to-end delivery of the mobile web experience, improving mobile retention by 22% in Q3 2021'
						]
					},
					{
						title: 'Frontend Developer',
						subtitle: 'Acme Agency',
						date: 'Jun 2018 – Feb 2020',
						location: 'New York, NY',
						description: '',
						details: [
							'Delivered 15+ client projects (e-commerce, marketing sites, internal tools) under tight deadlines with React and Vue',
							'Introduced automated testing (Jest + Playwright) that cut regression bugs by 40% across the client portfolio'
						]
					}
				]
			},
			{
				id: 'projects',
				title: 'Projects',
				type: 'projects',
				items: [
					{
						title: 'OpenMetrics',
						subtitle: 'github.com/alexjohnson/openmetrics',
						date: '2023',
						location: '',
						description: 'Open-source analytics dashboard built with SvelteKit and ClickHouse. 1.4k GitHub stars.',
						details: [
							'Handles 50M+ events/day with sub-second query latency via materialized views and aggressive caching',
							'Self-hostable via a single Docker Compose command; featured in Hacker News top 10'
						]
					},
					{
						title: 'Taskflow',
						subtitle: 'github.com/alexjohnson/taskflow',
						date: '2022',
						location: '',
						description: 'CLI task manager in TypeScript with a local-first sync engine.',
						details: [
							'Implemented a CRDT-based sync layer; zero data loss across offline/online state transitions'
						]
					}
				]
			},
			{
				id: 'education',
				title: 'Education',
				type: 'education',
				items: [
					{
						title: 'B.S. Computer Science',
						subtitle: 'University of California, Berkeley',
						date: '2014 – 2018',
						location: 'Berkeley, CA',
						description: '',
						details: []
					}
				]
			},
			{
				id: 'languages',
				title: 'Languages',
				type: 'languages',
				items: [
					{ name: 'English', level: 'Native' },
					{ name: 'Spanish', level: 'Conversational' }
				]
			}
		]
	};

	// ─── State ────────────────────────────────────────────────────────────────────
	let jsonText = $state(JSON.stringify(defaultCv, null, 2));
	let cv = $state(structuredClone(defaultCv));
	let showEditor = $state(false);
	let parseError = $state('');
	let copied = $state(false);

	// ─── Handlers ─────────────────────────────────────────────────────────────────
	function applyJson() {
		try {
			cv = JSON.parse(jsonText);
			parseError = '';
		} catch (e: any) {
			parseError = e.message;
		}
	}

	function handleFileUpload(e: Event) {
		const file = (e.target as HTMLInputElement).files?.[0];
		if (!file) return;
		const reader = new FileReader();
		reader.onload = () => {
			jsonText = reader.result as string;
			applyJson();
		};
		reader.readAsText(file);
	}

	async function exportPdf() {
		if (!browser) return;
		const html2pdf = (await import('html2pdf.js')).default;
		const el = document.getElementById('cv-render');
		if (!el) return;
		html2pdf()
			.set({
				margin: 0,
				filename: `${cv.personal.fullName.replace(/\s+/g, '_')}_CV.pdf`,
				image: { type: 'jpeg', quality: 0.98 },
				html2canvas: { scale: 2, useCORS: true },
				jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
			})
			.from(el)
			.save();
	}

	async function copyPrompt() {
		if (!browser) return;
		const schema = `{
  "personal": {
    "fullName": "string — full legal name",
    "title": "string — job title / professional headline",
    "email": "string",
    "phone": "string",
    "location": "string — City, Country",
    "summary": "string — 2-4 sentence professional summary, no first-person pronouns",
    "links": [{ "label": "string e.g. GitHub", "url": "string — short URL without https://" }]
  },
  "sections": [
    {
      "id": "skills",
      "title": "Technical Skills",
      "type": "skills",
      "items": [{ "category": "string e.g. Frontend", "values": ["string array of tools/techs"] }]
    },
    {
      "id": "experience",
      "title": "Work Experience",
      "type": "experience",
      "items": [{
        "title": "string — job title",
        "subtitle": "string — company name",
        "date": "string — e.g. Jan 2022 – Present",
        "location": "string — City, Country or Remote",
        "description": "string — optional one-liner (leave empty if using details)",
        "details": ["string array — bullet points, max 3-4 per role"]
      }]
    },
    {
      "id": "projects",
      "title": "Projects",
      "type": "projects",
      "items": [{
        "title": "string — project name",
        "subtitle": "string — URL or github link",
        "date": "string — year",
        "location": "",
        "description": "string — one-sentence description",
        "details": ["string array — bullet points"]
      }]
    },
    {
      "id": "education",
      "title": "Education",
      "type": "education",
      "items": [{
        "title": "string — degree name",
        "subtitle": "string — university/institution",
        "date": "string — year range",
        "location": "string",
        "description": "",
        "details": []
      }]
    },
    {
      "id": "languages",
      "title": "Languages",
      "type": "languages",
      "items": [{ "name": "string", "level": "Native | Fluent | Conversational | Basic" }]
    }
  ]
}`;

		const prompt = `You are a world-class CV writer and ATS optimization expert. Your job is to produce the best possible CV in a specific JSON format for a web-based CV builder.

## MODES

**Mode A — Interview Mode**
If the user says "ask me questions" or hasn't provided their info yet, interview them step by step. Ask about:
1. Target role and job description (paste it if they have one)
2. Work experience (company, title, dates, key achievements with metrics)
3. Technical skills and tools
4. Side projects and open-source work
5. Education and languages
After gathering all info, produce the final JSON.

**Mode B — Optimization Mode**
If the user pastes their existing CV, LinkedIn profile, raw notes, or a job description — analyze everything and produce a fully tailored, ATS-optimized JSON. If a job description is provided, weave relevant keywords naturally into the bullet points and skills section.

## JSON SCHEMA
${schema}

## WRITING RULES
- Bullet points must start with a strong past-tense action verb (Built, Led, Reduced, Shipped, Architected, Drove, Increased, Automated, Migrated, Designed, etc.)
- Every bullet should ideally include a metric or concrete outcome (%, $, users, time saved, team size)
- Keep each bullet to 1–2 lines max — tight and impactful
- Summary: 2–4 sentences, no "I" / "my", tailored to the target role
- Skills: group logically (Frontend, Backend, Tools, Cloud, etc.) — only include skills the person actually has
- Section order should be: skills → experience → projects → education → languages
- Only include sections that have real, non-empty content
- Links: short display form without "https://" (e.g. "github.com/username")
- Dates: consistent format e.g. "Jan 2022 – Present" or "2020 – 2022"

## OUTPUT FORMAT
Return ONLY the raw JSON object. No markdown code fences. No explanation. No commentary. Just the JSON, starting with { and ending with }.

---
Ready. Paste your CV / job description, or type "ask me questions" to begin.`;

		try {
			await navigator.clipboard.writeText(prompt);
			copied = true;
			setTimeout(() => (copied = false), 2000);
		} catch {
			// fallback for older browsers
			const ta = document.createElement('textarea');
			ta.value = prompt;
			ta.style.position = 'fixed';
			ta.style.opacity = '0';
			document.body.appendChild(ta);
			ta.select();
			document.execCommand('copy');
			document.body.removeChild(ta);
			copied = true;
			setTimeout(() => (copied = false), 2000);
		}
	}
</script>

<!-- ─── Toolbar ──────────────────────────────────────────────────────────────── -->
<div class="no-print sticky top-0 z-50 flex flex-wrap items-center gap-2 border-b border-neutral-200 bg-white px-4 py-2.5">
	<button
		onclick={() => (showEditor = !showEditor)}
		class="rounded border border-neutral-300 px-3 py-1.5 text-xs font-medium text-neutral-700 transition hover:bg-neutral-50"
	>
		{showEditor ? 'Hide Editor' : 'Edit JSON'}
	</button>

	<button
		onclick={copyPrompt}
		class="flex items-center gap-1.5 rounded border px-3 py-1.5 text-xs font-medium transition"
		class:border-green-400={copied}
		class:bg-green-50={copied}
		class:text-green-700={copied}
		class:border-neutral-300={!copied}
		class:text-neutral-700={!copied}
		class:hover:bg-neutral-50={!copied}
	>
		{#if copied}
			<svg xmlns="http://www.w3.org/2000/svg" class="h-3.5 w-3.5" viewBox="0 0 20 20" fill="currentColor">
				<path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd" />
			</svg>
			Copied!
		{:else}
			<svg xmlns="http://www.w3.org/2000/svg" class="h-3.5 w-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
				<path stroke-linecap="round" stroke-linejoin="round" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
			</svg>
			Copy AI Prompt
		{/if}
	</button>

	<button
		onclick={exportPdf}
		class="rounded bg-neutral-900 px-3 py-1.5 text-xs font-medium text-white transition hover:bg-neutral-700"
	>
		Export PDF
	</button>

	<label class="cursor-pointer rounded border border-neutral-300 px-3 py-1.5 text-xs font-medium text-neutral-700 transition hover:bg-neutral-50">
		Load JSON
		<input type="file" accept=".json" onchange={handleFileUpload} class="hidden" />
	</label>

	{#if parseError}
		<span class="text-xs text-red-500">{parseError}</span>
	{/if}
</div>

<!-- ─── JSON Editor ───────────────────────────────────────────────────────────── -->
{#if showEditor}
	<div class="no-print border-b border-neutral-200 bg-neutral-50 p-4">
		<p class="mb-2 text-xs text-neutral-400">Paste AI-generated JSON here and it renders live ↓</p>
		<textarea
			bind:value={jsonText}
			oninput={() => applyJson()}
			class="h-[42vh] w-full rounded border border-neutral-300 bg-white p-3 font-mono text-[11px] leading-relaxed focus:border-neutral-500 focus:outline-none"
			spellcheck="false"
		></textarea>
	</div>
{/if}

<!-- ─── CV Page ───────────────────────────────────────────────────────────────── -->
<div class="flex justify-center bg-neutral-200 py-10 print:bg-white print:py-0">
	<div
		id="cv-render"
		class="w-[210mm] min-h-[297mm] bg-white print:shadow-none print:w-full"
		style="
			font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
			box-shadow: 0 4px 32px rgba(0,0,0,0.12);
		"
	>
		<!-- Header ─────────────────────────────────────────────────── -->
		<header style="padding: 28px 40px 22px; border-bottom: 1px solid #e5e7eb;">
			<div style="display: flex; justify-content: space-between; align-items: flex-start; gap: 16px;">
				<!-- Name + Title -->
				<div>
					<div style="font-size: 22px; font-weight: 700; color: #0f172a; letter-spacing: -0.3px; line-height: 1.2;">
						{cv.personal.fullName}
					</div>
					<div style="font-size: 12.5px; font-weight: 500; color: #64748b; margin-top: 3px;">
						{cv.personal.title}
					</div>
				</div>
				<!-- Contact block -->
				<div style="text-align: right; font-size: 10.5px; color: #64748b; line-height: 1.7; flex-shrink: 0; margin-top: 2px;">
					{#if cv.personal.email}<div>{cv.personal.email}</div>{/if}
					{#if cv.personal.phone}<div>{cv.personal.phone}</div>{/if}
					{#if cv.personal.location}<div>{cv.personal.location}</div>{/if}
					{#each cv.personal.links as link}
						<div style="color: #334155; font-weight: 500;">{link.label} · {link.url}</div>
					{/each}
				</div>
			</div>
			<!-- Summary -->
			{#if cv.personal.summary}
				<p style="margin-top: 14px; font-size: 11px; line-height: 1.65; color: #475569; max-width: 148mm;">
					{cv.personal.summary}
				</p>
			{/if}
		</header>

		<!-- Body ────────────────────────────────────────────────────── -->
		<main style="padding: 18px 40px 32px;">
			{#each cv.sections as section, si}
				<div style="margin-top: {si === 0 ? '0' : '16px'};">
					<!-- Section heading -->
					<div style="
						font-size: 9px;
						font-weight: 700;
						text-transform: uppercase;
						letter-spacing: 0.1em;
						color: #94a3b8;
						padding-bottom: 5px;
						border-bottom: 1px solid #f1f5f9;
						margin-bottom: 9px;
					">
						{section.title}
					</div>

					<!-- Skills ─────────────────────────────────── -->
					{#if section.type === 'skills'}
						<div style="display: flex; flex-direction: column; gap: 3px;">
							{#each section.items as skill}
								<div style="display: flex; gap: 0; font-size: 11px; line-height: 1.55;">
									<span style="width: 100px; flex-shrink: 0; font-weight: 600; color: #374151;">{skill.category}</span>
									<span style="color: #6b7280;">{skill.values.join(', ')}</span>
								</div>
							{/each}
						</div>

					<!-- Experience / Education / Projects ───────── -->
					{:else if section.type === 'experience' || section.type === 'education' || section.type === 'projects'}
						<div style="display: flex; flex-direction: column; gap: 12px;">
							{#each section.items as item}
								<div>
									<!-- Item header row -->
									<div style="display: flex; justify-content: space-between; align-items: baseline; gap: 8px;">
										<div style="display: flex; align-items: baseline; gap: 0; flex-wrap: wrap; min-width: 0;">
											<span style="font-size: 11.5px; font-weight: 700; color: #1e293b;">{item.title}</span>
											{#if item.subtitle}
												<span style="font-size: 11px; color: #64748b; margin-left: 6px;">— {item.subtitle}</span>
											{/if}
										</div>
										<div style="font-size: 10.5px; color: #94a3b8; flex-shrink: 0; white-space: nowrap;">
											{#if item.date && item.location}
												{item.date} · {item.location}
											{:else if item.date}
												{item.date}
											{:else if item.location}
												{item.location}
											{/if}
										</div>
									</div>
									<!-- Description -->
									{#if item.description}
										<p style="margin: 3px 0 0; font-size: 11px; color: #64748b; line-height: 1.5;">{item.description}</p>
									{/if}
									<!-- Bullet details -->
									{#if item.details?.length}
										<div style="margin-top: 4px; display: flex; flex-direction: column; gap: 2.5px;">
											{#each item.details as detail}
												<div style="display: flex; gap: 8px; font-size: 11px; line-height: 1.55; color: #374151;">
													<span style="flex-shrink: 0; margin-top: 5.5px; width: 3px; height: 3px; border-radius: 50%; background: #9ca3af; display: block;"></span>
													<span>{detail}</span>
												</div>
											{/each}
										</div>
									{/if}
								</div>
							{/each}
						</div>

					<!-- Languages ───────────────────────────────── -->
					{:else if section.type === 'languages'}
						<div style="display: flex; gap: 24px; flex-wrap: wrap;">
							{#each section.items as lang}
								<div style="font-size: 11px;">
									<span style="font-weight: 600; color: #374151;">{lang.name}</span>
									<span style="color: #9ca3af; margin-left: 4px;">({lang.level})</span>
								</div>
							{/each}
						</div>
					{/if}
				</div>
			{/each}
		</main>
	</div>
</div>
