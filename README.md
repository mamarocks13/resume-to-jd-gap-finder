# resume-to-jd-gap-finder
Compares resume to a job description and flags missing skills

import re
from collections import Counter

# You can expand this list over time based on the kinds of jobs you want
KNOWN_SKILLS = [
    # Technical / data
    "python", "sql", "excel", "tableau", "power bi", "looker", "jira", "salesforce",
    "api", "apis", "json", "csv", "machine learning", "artificial intelligence",
    "ai", "llm", "rag", "automation", "data analysis", "data analytics",
    "forecasting", "financial modeling", "dashboard", "dashboards",

    # Strategy / operations
    "strategy", "strategic planning", "operations", "business operations",
    "program management", "project management", "process improvement",
    "change management", "roadmap", "roadmaps", "execution", "planning",
    "stakeholder management", "cross-functional", "cross functional",
    "kpi", "kpis", "okr", "okrs", "go-to-market", "gtm",

    # Commercial / business
    "pricing", "merchandising", "category management", "p&l", "profitability",
    "growth", "customer insights", "consumer insights", "market research",
    "experimentation", "a/b testing", "retail", "ecommerce", "e-commerce",
    "supply chain", "logistics", "last mile", "vendor management",

    # Leadership
    "leadership", "team leadership", "people management", "communication",
    "presentation", "executive communication", "problem solving"
]

STOPWORDS = {
    "the", "and", "or", "a", "an", "to", "of", "for", "in", "on", "at", "by",
    "with", "from", "is", "are", "was", "were", "be", "been", "being", "as",
    "that", "this", "these", "those", "it", "its", "will", "can", "may",
    "our", "we", "you", "your", "their", "they", "them", "but", "if", "then",
    "than", "so", "such", "into", "about", "over", "under", "across", "through",
    "who", "what", "when", "where", "how", "all", "any", "each", "other",
    "more", "most", "some", "no", "not", "only", "job", "role", "position",
    "team", "company", "candidate", "candidates", "experience", "skills",
    "skill", "ability", "abilities", "work", "working", "responsible",
    "responsibilities", "requirements", "required", "preferred", "plus"
}


def read_text_file(file_path):
    with open(file_path, "r", encoding="utf-8") as file:
        return file.read()


def normalize_text(text):
    text = text.lower()
    text = re.sub(r"[^a-z0-9\s\-/&]", " ", text)
    text = re.sub(r"\s+", " ", text).strip()
    return text


def extract_known_skills(text, known_skills):
    found_skills = {}
    for skill in known_skills:
        # Allow flexible match for hyphens/spaces
        pattern_text = re.escape(skill.lower())
        pattern_text = pattern_text.replace(r"\ ", r"[\s\-]+")
        pattern = rf"\b{pattern_text}\b"
        matches = re.findall(pattern, text)
        if matches:
            found_skills[skill] = len(matches)
    return found_skills


def tokenize(text):
    words = text.split()
    return [w for w in words if w not in STOPWORDS and len(w) > 2]


def extract_top_keywords(text, top_n=25):
    tokens = tokenize(text)
    counts = Counter(tokens)
    return counts.most_common(top_n)


def compare_resume_to_job(resume_text, job_text, known_skills):
    normalized_resume = normalize_text(resume_text)
    normalized_job = normalize_text(job_text)

    resume_skills = extract_known_skills(normalized_resume, known_skills)
    job_skills = extract_known_skills(normalized_job, known_skills)

    missing_skills = sorted(set(job_skills.keys()) - set(resume_skills.keys()))
    overlapping_skills = sorted(set(job_skills.keys()) & set(resume_skills.keys()))

    return {
        "resume_skills": resume_skills,
        "job_skills": job_skills,
        "missing_skills": missing_skills,
        "overlapping_skills": overlapping_skills,
        "top_job_keywords": extract_top_keywords(normalized_job),
        "top_resume_keywords": extract_top_keywords(normalized_resume),
    }


def print_report(results):
    print("\n" + "=" * 60)
    print("RESUME VS JOB DESCRIPTION SKILL GAP REPORT")
    print("=" * 60)

    print("\nSkills found in job description:")
    if results["job_skills"]:
        for skill, count in sorted(results["job_skills"].items(), key=lambda x: x[1], reverse=True):
            print(f"  - {skill} ({count})")
    else:
        print("  None found from the known skills list.")

    print("\nSkills found in resume:")
    if results["resume_skills"]:
        for skill, count in sorted(results["resume_skills"].items(), key=lambda x: x[1], reverse=True):
            print(f"  - {skill} ({count})")
    else:
        print("  None found from the known skills list.")

    print("\nMatching skills already in resume:")
    if results["overlapping_skills"]:
        for skill in results["overlapping_skills"]:
            print(f"  - {skill}")
    else:
        print("  None")

    print("\nLikely missing skills:")
    if results["missing_skills"]:
        for skill in results["missing_skills"]:
            print(f"  - {skill}")
    else:
        print("  None — your resume appears to contain the job's known skills.")

    print("\nTop keywords in job description:")
    for word, count in results["top_job_keywords"][:15]:
        print(f"  - {word} ({count})")

    print("\nTop keywords in resume:")
    for word, count in results["top_resume_keywords"][:15]:
        print(f"  - {word} ({count})")


def main():
    resume_file = "resume.txt"
    job_file = "job_description.txt"

    resume_text = read_text_file(resume_file)
    job_text = read_text_file(job_file)

    results = compare_resume_to_job(resume_text, job_text, KNOWN_SKILLS)
    print_report(results)


if __name__ == "__main__":
    main()
