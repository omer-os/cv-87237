<script lang="ts">
	import { browser } from '$app/environment';

	// ─── Placeholder data — replace via "Copy AI Prompt" workflow ────────────────
	const defaultCv = {
		personal: {
			fullName: 'Alex Johnson',
			title: 'Senior Frontend Engineer',
			email: 'alex@example.com',
			phone: '+1 (415) 555-0192',
			location: 'San Francisco, CA',
			summary:
				'Frontend engineer with 5 years shipping production web apps at scale. Deep expertise in React, TypeScript, and modern build tooling with a track record of reducing load times, leading component system migrations, and owning cross-functional delivery end-to-end.',
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
					{ category: 'Frontend', values: ['React', 'Next.js', 'TypeScript', 'SvelteKit', 'TailwindCSS', 'Zustand', 'React Query'] },
					{ category: 'Backend & Data', values: ['Node.js', 'PostgreSQL', 'Redis', 'Prisma', 'REST APIs', 'GraphQL'] },
					{ category: 'Tooling', values: ['Vite', 'Webpack', 'Docker', 'GitHub Actions', 'AWS', 'Figma'] }
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
							'Led rebuild of the merchant dashboard in Next.js App Router — reduced LCP by 52% and cut support tickets related to UI by 30%',
							'Designed and shipped a 60-component library adopted across 8 product teams, eliminating duplicate UI work and standardising design tokens',
							'Built real-time transaction monitoring with WebSockets + React Query serving 200k+ merchants; zero downtime migrations across 3 major releases'
						]
					},
					{
						title: 'Frontend Engineer',
						subtitle: 'Notion',
						date: 'Mar 2020 – Dec 2021',
						location: 'Remote',
						description: '',
						details: [
							'Shipped the collaborative editing toolbar now used by 30M+ users; implemented conflict-free real-time sync with zero merge-loss bugs post-launch',
							'Migrated legacy class components to Hooks + Context, shrinking bundle by 18% and improving TTI on low-end devices by 400ms',
							'Owned the mobile web experience end-to-end — improved mobile retention 22% in Q3 2021 after redesigning navigation and scroll performance'
						]
					},
					{
						title: 'Frontend Developer',
						subtitle: 'Acme Agency',
						date: 'Jun 2018 – Feb 2020',
						location: 'New York, NY',
						description: '',
						details: [
							'Delivered 15+ client projects (e-commerce, portals, admin tools) in React and Vue under fixed-price contracts with 100% on-time delivery rate',
							'Introduced Jest + Playwright test coverage from 0% to 70%, reducing regression bugs by 40% across the entire client portfolio'
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
						description: 'Self-hostable product analytics in SvelteKit + ClickHouse. 1.4k GitHub stars, featured in Hacker News top 10.',
						details: [
							'Handles 50M+ events/day at sub-second query latency via materialized views and query caching',
							'Single-command Docker Compose deploy — 300+ production installs within first month of release'
						]
					},
					{
						title: 'Taskflow CLI',
						subtitle: 'github.com/alexjohnson/taskflow',
						date: '2022',
						location: '',
						description: 'Local-first task manager in TypeScript with CRDT-based sync — zero data loss across offline/online transitions.',
						details: []
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
	let exporting = $state(false);

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
		if (!browser || exporting) return;
		exporting = true;
		const { default: html2pdf } = await import('html2pdf.js');
		const el = document.getElementById('cv-render') as HTMLElement;
		if (!el) { exporting = false; return; }

		// Temporarily clamp to exact A4 height so html2pdf captures exactly one page
		const prev = { h: el.style.height, oh: el.style.overflow, mh: el.style.minHeight };
		el.style.height = '297mm';
		el.style.minHeight = '0';
		el.style.overflow = 'hidden';

		try {
			await html2pdf()
				.set({
					margin: 0,
					filename: `${cv.personal.fullName.replace(/\s+/g, '_')}_CV.pdf`,
					image: { type: 'jpeg', quality: 0.98 },
					html2canvas: { scale: 2, useCORS: true, logging: false },
					jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
				})
				.from(el)
				.save();
		} finally {
			el.style.height = prev.h;
			el.style.minHeight = prev.mh;
			el.style.overflow = prev.oh;
			exporting = false;
		}
	}

	async function copyPrompt() {
		if (!browser) return;

		const schema = `{
  "personal": {
    "fullName": "string",
    "title": "string — concise job title / professional headline",
    "email": "string",
    "phone": "string",
    "location": "string — City, Country",
    "summary": "string — 2-3 sentences, NO first-person pronouns, NO filler phrases like 'passionate about' or 'dynamic team player'. Start with role + years + core strength. Tailored to the target role.",
    "links": [
      { "label": "GitHub", "url": "github.com/username" },
      { "label": "LinkedIn", "url": "linkedin.com/in/username" }
    ]
  },
  "sections": [
    {
      "id": "skills",
      "title": "Technical Skills",
      "type": "skills",
      "items": [
        { "category": "Frontend", "values": ["React", "TypeScript", "..."] },
        { "category": "Backend", "values": ["Node.js", "PostgreSQL", "..."] },
        { "category": "Tooling", "values": ["Git", "Docker", "..."] }
      ]
    },
    {
      "id": "experience",
      "title": "Work Experience",
      "type": "experience",
      "items": [
        {
          "title": "Job Title",
          "subtitle": "Company Name",
          "date": "Mon YYYY – Mon YYYY or Present",
          "location": "City, Country or Remote",
          "description": "",
          "details": [
            "3-4 bullet points per role MAX. Each must: start with a strong past-tense action verb, include a concrete metric or outcome, be 1-2 lines. NO generic duties."
          ]
        }
      ]
    },
    {
      "id": "projects",
      "title": "Projects",
      "type": "projects",
      "items": [
        {
          "title": "Project Name",
          "subtitle": "github.com/... or live URL",
          "date": "YYYY",
          "location": "",
          "description": "One-line description with tech stack and scale/impact.",
          "details": ["Optional: 1-2 bullets for standout technical decisions or metrics"]
        }
      ]
    },
    {
      "id": "education",
      "title": "Education",
      "type": "education",
      "items": [
        {
          "title": "Degree Name",
          "subtitle": "University Name",
          "date": "YYYY – YYYY",
          "location": "City, Country",
          "description": "",
          "details": []
        }
      ]
    },
    {
      "id": "languages",
      "title": "Languages",
      "type": "languages",
      "items": [
        { "name": "English", "level": "Native" }
      ]
    }
  ]
}`;

		const prompt = `You are a senior technical recruiter and elite CV writer. Your output gets candidates interviews at top-tier companies. You produce CVs that are tight, honest, metric-driven, and ATS-optimised — zero fluff, zero AI slop.

## YOUR TASK
Produce the best possible CV as a JSON object matching the schema below.

## TWO MODES

**Mode A — Interview:** If the user says "ask me questions", interview them step by step:
1. Target role + job description (have them paste it)
2. Each work experience: company, title, dates, the actual work done, any metrics you can extract (%, $, users, time, team size)
3. Side projects and open-source
4. Skills (only what they genuinely know)
5. Education + languages
After all info is gathered, output the final JSON.

**Mode B — Optimize:** If the user pastes a CV, resume, LinkedIn dump, or job description — analyse it all and produce a polished, ATS-optimised JSON. If a job description is provided, mirror its exact keywords and phrases in the skills section and naturally in bullet points. Do not invent experience — only reframe and sharpen what exists.

## JSON SCHEMA
${schema}

## HARD RULES FOR BULLET POINTS
- Every bullet starts with a strong PAST-TENSE action verb: Built, Led, Reduced, Increased, Shipped, Architected, Migrated, Automated, Designed, Owned, Drove, Launched, Refactored, Scaled, Integrated
- Every bullet includes at least one concrete number, %, $, or user count where possible
- Maximum 3-4 bullets per role — only the highest-impact ones
- No bullet longer than 2 lines
- BANNED phrases: "responsible for", "worked on", "helped with", "assisted in", "passionate about", "team player", "fast learner", "detail-oriented", "leverage", "utilize", "dynamic", "synergy"

## HARD RULES FOR SUMMARY
- 2-3 sentences maximum
- No "I" / "my" / "me"
- No "passionate", "motivated", "results-driven", "dynamic"
- Structure: [Role] with [X years] of [specific expertise]. [Strongest concrete achievement or specialisation]. [What they bring to the target role — specific, not generic.]

## ATS RULES
- Section titles must be standard: "Work Experience", "Technical Skills", "Education", "Projects", "Languages"
- Skills: only include what the candidate genuinely knows. Group into 3-4 logical categories.
- Links: short display URLs without "https://" (e.g. "github.com/username")
- Dates: consistent format e.g. "Jan 2022 – Present"

## OUTPUT FORMAT
Return ONLY the raw JSON object. No markdown. No \`\`\`json. No explanation. No commentary before or after. Start with { and end with }.

---
Ready. Paste your info, existing CV, or job description — or say "ask me questions" to start the interview.`;

		try {
			await navigator.clipboard.writeText(prompt);
		} catch {
			const ta = document.createElement('textarea');
			ta.value = prompt;
			Object.assign(ta.style, { position: 'fixed', opacity: '0', top: '0', left: '0' });
			document.body.appendChild(ta);
			ta.select();
			document.execCommand('copy');
			document.body.removeChild(ta);
		}
		copied = true;
		setTimeout(() => (copied = false), 2200);
	}
</script>

<!-- ─── Toolbar ──────────────────────────────────────────────────────────────── -->
<div class="no-print sticky top-0 z-50 flex flex-wrap items-center gap-2 border-b border-neutral-200 bg-white px-4 py-2.5 shadow-sm">
	<span class="mr-1 text-xs font-semibold text-neutral-400 tracking-wide">CV BUILDER</span>
	<div class="h-4 w-px bg-neutral-200 mx-1"></div>

	<button
		onclick={() => (showEditor = !showEditor)}
		class="rounded border border-neutral-300 px-3 py-1.5 text-xs font-medium text-neutral-700 transition hover:bg-neutral-50"
	>
		{showEditor ? 'Hide Editor' : 'Edit JSON'}
	</button>

	<button
		onclick={copyPrompt}
		class="flex items-center gap-1.5 rounded border px-3 py-1.5 text-xs font-medium transition"
		style={copied ? 'border-color:#4ade80;background:#f0fdf4;color:#15803d;' : 'border-color:#d1d5db;color:#374151;'}
	>
		{#if copied}
			<svg xmlns="http://www.w3.org/2000/svg" class="h-3.5 w-3.5" viewBox="0 0 20 20" fill="currentColor">
				<path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd" />
			</svg>
			Prompt copied!
		{:else}
			<svg xmlns="http://www.w3.org/2000/svg" class="h-3.5 w-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
				<path stroke-linecap="round" stroke-linejoin="round" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
			</svg>
			Copy AI Prompt
		{/if}
	</button>

	<button
		onclick={exportPdf}
		disabled={exporting}
		class="rounded bg-neutral-900 px-3 py-1.5 text-xs font-medium text-white transition hover:bg-neutral-700 disabled:opacity-50"
	>
		{exporting ? 'Exporting…' : 'Export PDF'}
	</button>

	<label class="cursor-pointer rounded border border-neutral-300 px-3 py-1.5 text-xs font-medium text-neutral-700 transition hover:bg-neutral-50">
		Load JSON
		<input type="file" accept=".json" onchange={handleFileUpload} class="hidden" />
	</label>

	{#if parseError}
		<span class="text-xs text-red-500 ml-1">{parseError}</span>
	{/if}
</div>

<!-- ─── JSON Editor ───────────────────────────────────────────────────────────── -->
{#if showEditor}
	<div class="no-print border-b border-neutral-200 bg-neutral-50 p-4">
		<p class="mb-2 text-xs text-neutral-400">
			1. Click <strong class="text-neutral-600">Copy AI Prompt</strong> → paste into Claude / ChatGPT →
			get JSON back → paste here → CV renders live below.
		</p>
		<textarea
			bind:value={jsonText}
			oninput={() => applyJson()}
			class="h-[44vh] w-full rounded border border-neutral-300 bg-white p-3 font-mono text-[11px] leading-relaxed focus:border-neutral-500 focus:outline-none"
			spellcheck="false"
		></textarea>
	</div>
{/if}

<!-- ─── Page background ───────────────────────────────────────────────────────── -->
<div class="flex justify-center bg-neutral-300 py-10 print:bg-white print:py-0">
	<!--
		min-h keeps it A4-tall on screen.
		During export, JS clamps this to exactly 297mm + overflow:hidden = single page.
	-->
	<div
		id="cv-render"
		style="
			width: 210mm;
			min-height: 297mm;
			background: #ffffff;
			font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
			box-shadow: 0 2px 40px rgba(0,0,0,0.18);
		"
	>
		<!-- ── Header ─────────────────────────────────────────────────────────── -->
		<div style="padding: 26px 38px 20px 38px; border-bottom: 1.5px solid #e2e8f0;">
			<!-- Row 1: Name left, contact right -->
			<div style="display:flex; justify-content:space-between; align-items:flex-start; gap:12px;">
				<div>
					<div style="font-size:21px; font-weight:800; color:#0f172a; letter-spacing:-0.4px; line-height:1.15;">
						{cv.personal.fullName}
					</div>
					<div style="font-size:12px; font-weight:500; color:#64748b; margin-top:3px; letter-spacing:0.01em;">
						{cv.personal.title}
					</div>
				</div>
				<div style="text-align:right; font-size:10px; color:#64748b; line-height:1.8; flex-shrink:0; padding-top:1px;">
					{#if cv.personal.email}<div>{cv.personal.email}</div>{/if}
					{#if cv.personal.phone}<div>{cv.personal.phone}</div>{/if}
					{#if cv.personal.location}<div>{cv.personal.location}</div>{/if}
					{#each cv.personal.links as link}
						<div style="font-weight:600; color:#334155;">{link.label} · {link.url}</div>
					{/each}
				</div>
			</div>
			<!-- Summary -->
			{#if cv.personal.summary}
				<p style="margin:12px 0 0; font-size:10.5px; line-height:1.65; color:#475569; max-width:150mm;">
					{cv.personal.summary}
				</p>
			{/if}
		</div>

		<!-- ── Sections ───────────────────────────────────────────────────────── -->
		<div style="padding: 16px 38px 28px 38px;">
			{#each cv.sections as section, si}
				<div style="margin-top: {si === 0 ? '0' : '14px'};">

					<!-- Section label -->
					<div style="
						font-size: 8.5px;
						font-weight: 800;
						text-transform: uppercase;
						letter-spacing: 0.13em;
						color: #94a3b8;
						padding-bottom: 4px;
						border-bottom: 1px solid #e2e8f0;
						margin-bottom: 8px;
					">
						{section.title}
					</div>

					<!-- ── Skills ──────────────────────────────────────────────── -->
					{#if section.type === 'skills'}
						<div style="display:flex; flex-direction:column; gap:2.5px;">
							{#each section.items as skill}
								<div style="display:flex; font-size:10.5px; line-height:1.5;">
									<span style="width:92px; flex-shrink:0; font-weight:700; color:#1e293b;">{skill.category}</span>
									<span style="color:#64748b;">{skill.values.join(', ')}</span>
								</div>
							{/each}
						</div>

					<!-- ── Experience / Education / Projects ────────────────────── -->
					{:else if section.type === 'experience' || section.type === 'education' || section.type === 'projects'}
						<div style="display:flex; flex-direction:column; gap:10px;">
							{#each section.items as item}
								<div>
									<!-- Item title row -->
									<div style="display:flex; justify-content:space-between; align-items:baseline; gap:8px;">
										<div style="font-size:11px; font-weight:700; color:#0f172a; line-height:1.3;">
											{item.title}
											{#if item.subtitle}
												<span style="font-weight:400; color:#64748b; margin-left:5px; font-size:10.5px;">— {item.subtitle}</span>
											{/if}
										</div>
										<div style="font-size:10px; color:#94a3b8; flex-shrink:0; white-space:nowrap;">
											{[item.date, item.location].filter(Boolean).join(' · ')}
										</div>
									</div>
									<!-- Optional one-liner description -->
									{#if item.description}
										<p style="margin:2px 0 0; font-size:10.5px; line-height:1.55; color:#64748b;">{item.description}</p>
									{/if}
									<!-- Bullet points — text-indent hanging indent for reliable PDF rendering -->
									{#if item.details?.length}
										<div style="margin-top:3px; display:flex; flex-direction:column; gap:2px;">
											{#each item.details as detail}
												<p style="
													margin:0;
													font-size:10.5px;
													line-height:1.6;
													color:#334155;
													padding-left:12px;
													text-indent:-12px;
												">
													<span style="color:#94a3b8; font-size:10px;">–&nbsp;</span>{detail}
												</p>
											{/each}
										</div>
									{/if}
								</div>
							{/each}
						</div>

					<!-- ── Languages ─────────────────────────────────────────────── -->
					{:else if section.type === 'languages'}
						<div style="display:flex; flex-wrap:wrap; gap:20px;">
							{#each section.items as lang}
								<div style="font-size:10.5px;">
									<span style="font-weight:700; color:#1e293b;">{lang.name}</span>
									<span style="color:#94a3b8; margin-left:3px;">({lang.level})</span>
								</div>
							{/each}
						</div>
					{/if}

				</div>
			{/each}
		</div>
	</div>
</div>
