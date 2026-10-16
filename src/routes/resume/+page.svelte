<svelte:head>
    <title>Resume</title>
</svelte:head>

<h1 class="resumeHeading">Resume</h1>
<p class="contact">
    <a href="mailto:omdias77@mit.edu">omdias77@mit.edu</a>
    · <a href="tel:508-208-4413">508-208-4413</a>
    · <a href="https://github.com/Livy456" target="_blank" rel="noreferrer">GitHub</a>
    · <a href="https://www.linkedin.com/in/olivia-dias/" target="_blank" rel="noreferrer">LinkedIn</a>
</p>

<div class="resume">
    <section class="Education">
        <h3>EDUCATION</h3>
        <p><strong>Massachusetts Institute of Technology (MIT) Media Lab</strong>,
            MEng in Computer Science and Artificial Intelligence, Cambridge, MA
            <br><time>Graduated May 2026</time></p>
        <p><strong>Massachusetts Institute of Technology (MIT)</strong>,
            B.S. in Computer Science and Artificial Intelligence, Cambridge, MA
            <br><time>Graduated May 2025</time></p>
        <p><strong>WallStreetQuants</strong>, Online Course
            <br><time>August 2026 – present</time></p>
    </section>

    <section class="Technical_Skills">
        <h3>TECHNICAL SKILLS</h3>
        <p><strong>AI &amp; LLMs:</strong> LLM APIs (OpenAI GPT-5.4, Anthropic Claude Opus), RAG, agentic workflows, prompt orchestration, Whisper, LLM evaluation</p>
        <p><strong>Engineering &amp; Data:</strong> Python (pandas, NumPy), SQL, TypeScript, FastAPI, REST APIs, PostgreSQL, data pipelines, React</p>
        <p><strong>Production &amp; Security:</strong> AWS, Docker, CI/CD (GitHub Actions), integration testing, HTTPS/TLS, Linux, data governance (FERPA)</p>
        <p><strong>Finance &amp; Analytics:</strong> Time series modeling, Ornstein-Uhlenbeck, regression, statistical testing, settlement risk</p>
    </section>

    <section class="Relevant_Experience">
        <h3>RESEARCH &amp; PROFESSIONAL EXPERIENCE</h3>
        <p><strong>MIT Media Lab, Personal Robots Group</strong>, <em>Full Stack Software Developer I</em>, Cambridge, MA
            <br><time>July 2026 – Present</time></p>
        <ul>
            <li>Communicated with business stakeholders, researchers, and an executive as Technology Strategy Lead, running weekly sessions with a 6 person team to surface the highest value problems, coordinate user testing and adoption, and report to leadership.</li>
            <li>Built and shipped an LLM-powered review platform end to end, from data pipeline to user facing website, that ingests thousands of video, code, and text submissions, transcribes audio with Whisper, analyzes them with GPT-5.4 and Claude Opus, and escalates low-confidence cases, automating ~75% of a manual review workflow.</li>
            <li>Measured impact before scaling by benchmarking AI output against 95 human-scored cases (Wilcoxon, ICC, regression); caught systematic underscoring of video, fixed the routing logic, and published the evaluation at AHFE International proceedings.</li>
            <li>Made the system production-grade and compliant: deployed on AWS (FastAPI, PostgreSQL, Docker, HTTPS/TLS) with single-command releases, and designing FERPA-compliant data governance and partner data sharing agreements.</li>
        </ul>

        <p><strong>Nasdaq</strong>, <em>Data Scientist and Deep Learning Engineer</em>, Boston, MA
            <br><time>June 2025 – Aug 2025</time></p>
        <ul>
            <li>Built an Ornstein-Uhlenbeck mean reversion model (15 day lookback) predicting daily closes for 10 Nasdaq-listed companies, reducing RMSE by 6% vs. a rolling mean baseline in a walk forward test with no look ahead bias.</li>
            <li>Modeled NYSE-listed company performance after a Nasdaq transition (R² = 0.79), quantifying statistically significant liquidity gains and translating them into insights that supported Nasdaq’s pitch for multi-billion dollar listings from rival exchanges.</li>
        </ul>

        <p><strong>Nasdaq</strong>, <em>Machine Learning Engineer</em>, Boston, MA
            <br><time>Jan 2025</time></p>
        <ul>
            <li>Built “SettleGuard,” a proof of concept to automate settlement oversight, engineering features from raw Parquet banking data and tuning a classifier to 70% recall, deliberately prioritizing catching failures on high value transactions.</li>
        </ul>

        <p><strong>Procter &amp; Gamble (P&amp;G)</strong>, <em>Consumer Market Knowledge Business Analyst / Data Scientist</em>, Cincinnati, OH
            <br><time>June 2024 – Aug 2024</time></p>
        <ul>
            <li>Modeled drivers of media reach for Pampers, Always, Downy, and Oral-B using reach curves, regression, moving averages, and polynomial trend fitting on Meta and The Trade Desk data, quantifying the effects of frequency, cost, impressions, and targeting accuracy.</li>
            <li>Modeled how sales, eContent, bid amounts, velocity, and search terms drive organic rank and share of search, challenging the assumption that keyword inclusion mattered most: term placement had the larger effect.</li>
        </ul>
    </section>

    <section class="Community_Leadership">
        <h3>MIT LEADERSHIP &amp; ACTIVITIES</h3>
        <ul>
            <li>Captained MIT Track and Field (120 athletes) to DIII indoor and outdoor national titles; 3x All-American.</li>
            <li>Served as NSBE Membership Chair and Athletes In Action Group Leader.</li>
        </ul>
    </section>
</div>

<style>
    .contact {
        margin: -0.35rem 0 1.25rem;
        text-align: center;
    }

    .contact a {
        color: var(--accent-strong, steelblue);
    }

    .resume ul {
        margin: 0.15rem 0 0.9rem;
        padding-left: 1.35rem;
    }

    .resume li {
        margin: 0.35rem 0;
    }
</style>
