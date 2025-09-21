# Skillora-
Skillora is a smart platform for evaluating skills and offering personalized insights. It helps learners, freelancers, and job seekers grow through intuitive assessments and clear feedback. Designed for simplicity and impact, Skillora promotes self-improvement without technical complexity.
Next.js‎# SkillInsight — Full Next.js App (Deploy-ready for Vercel)
‎
‎This repository contains a production-ready Next.js application (React) named **SkillInsight** — a skill assessment platform with user auth, dynamic tests, evaluation logic (server-side), performance summaries, downloadable PDF certificates, and an admin dashboard. All user-facing labels avoid the term "AI" and instead use "Smart Evaluation", "Skill Insight", "Intelligent Feedback", etc.
‎
‎---
‎
‎## Project structure (high level)
‎
‎```
‎skillinsight/
‎├─ app/                       # Next.js App Router pages + layout
‎│  ├─ layout.jsx
‎│  ├─ page.jsx                # Landing
‎│  ├─ dashboard/              # User dashboard and tests
‎│  │  ├─ page.jsx
‎│  │  ├─ test/[id]/page.jsx
‎│  │  └─ results/[id]/page.jsx
‎│  ├─ auth/
‎│  │  ├─ login/page.jsx
‎│  │  └─ register/page.jsx
‎│  └─ admin/
‎│     ├─ page.jsx
‎│     ├─ tests/page.jsx
‎│     └─ tests/[id]/page.jsx
‎├─ components/
‎│  ├─ Navbar.jsx
‎│  ├─ ProtectedRoute.jsx
‎│  ├─ QuestionRenderer.jsx
‎│  ├─ ResultSummary.jsx
‎│  └─ CertificateModal.jsx
‎├─ lib/
‎│  ├─ supabaseClient.js
‎│  └─ evaluation.js           # Backend evaluation helpers (also used in /api)
‎├─ pages/api/
‎│  ├─ evaluate.js             # Server-side evaluation endpoint
‎│  └─ admin/tests.js          # Admin CRUD (serverless)
‎├─ public/
‎│  └─ assets/
‎├─ styles/
‎│  └─ globals.css
‎├─ utils/
‎│  └─ pdf.js                  # client-side certificate generation helper
‎├─ sql/
‎│  └─ supabase_schema.sql     # schema for Supabase
‎├─ package.json
‎├─ tailwind.config.js
‎├─ postcss.config.js
‎├─ next.config.js
‎├─ vercel.json
‎└─ README.md
‎```
‎
‎---
‎
‎## Key design choices
‎
‎- **Next.js (App Router)** with serverless API endpoints for evaluation and admin operations.
‎- **Supabase** for auth and database (simple to replace with Firebase if preferred).
‎- **Tailwind CSS** for responsive, mobile-friendly UI.
‎- **Client-side PDF generation** using `html2canvas` + `jspdf` (downloadable certificate). Serverless endpoints only do evaluation/analytics.
‎- **Evaluation logic**: deterministic embedded algorithms (keyword scoring, fuzzy matching, weighted scoring) implemented in `lib/evaluation.js` and used by `/api/evaluate`. No references to "AI" in UI.
‎
‎---
‎
‎## Environment variables (set in Vercel dashboard or .env.local)
‎
‎```
‎NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
‎NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
‎SUPABASE_SERVICE_ROLE_KEY=your_service_role_key  # only for server-side calls in Vercel env
‎NEXT_PUBLIC_APP_NAME=SkillInsight
‎```
‎
‎> On Vercel, set `SUPABASE_SERVICE_ROLE_KEY` as a server-only environment variable.
‎
‎---
‎
‎## Database schema (supabase_schema.sql)
‎
‎```sql
‎-- sql/supabase_schema.sql
‎
‎-- Users table comes built-in with Supabase Auth; additional profile data:
‎create table profiles (
‎  id uuid references auth.users not null primary key,
‎  full_name text,
‎  role text default 'user', -- 'user' or 'admin'
‎  created_at timestamptz default now()
‎);
‎
‎-- Tests and questions
‎create table tests (
‎  id uuid primary key default uuid_generate_v4(),
‎  title text not null,
‎  description text,
‎  created_by uuid references auth.users,
‎  created_at timestamptz default now()
‎);
‎
‎create table questions (
‎  id uuid primary key default uuid_generate_v4(),
‎  test_id uuid references tests(id) on delete cascade,
‎  type text not null, -- 'mcq' or 'short'
‎  text text not null,
‎  choices jsonb,       -- [{"id": 1, "text": "A"}, ...] for mcq
‎  answer jsonb,        -- correct answer (for mcq it's choice id; for short it's keywords)
‎  points int default 1
‎);
‎
‎-- User attempts & answers
‎create table attempts (
‎  id uuid primary key default uuid_generate_v4(),
‎  user_id uuid references auth.users,
‎  test_id uuid references tests(id),
‎  started_at timestamptz default now(),
‎  finished_at timestamptz,
‎  score numeric,
‎  details jsonb -- per-question results
‎);
‎
‎-- Simple analytics view can be derived with queries
‎```
‎
‎---
‎
‎## package.json (key deps)
‎
‎```json
‎{
‎  "name": "skillinsight",
‎  "version": "1.0.0",
‎  "private": true,
‎  "scripts": {
‎    "dev": "next dev",
‎    "build": "next build",
‎    "start": "next start",
‎    "lint": "next lint"
‎  },
‎  "dependencies": {
‎    "next": "14.0.0",
‎    "react": "18.2.0",
‎    "react-dom": "18.2.0",
‎    "@supabase/supabase-js": "2.0.0",
‎    "tailwindcss": "^3.0.0",
‎    "autoprefixer": "^10.0.0",
‎    "postcss": "^8.0.0",
‎    "jspdf": "^2.5.1",
‎    "html2canvas": "^1.4.1",
‎    "uuid": "^9.0.0",
‎    "lodash": "^4.17.21"
‎  }
‎}
‎```
‎
‎---
‎
‎## Vercel config (vercel.json)
‎
‎```json
‎{
‎  "version": 2,
‎  "builds": [
‎    { "src": "next.config.js", "use": "@vercel/next" }
‎  ],
‎  "routes": [
‎    { "src": "/api/(.*)", "dest": "/api/$1" }
‎  ]
‎}
‎```
‎
‎---
‎
‎## Tailwind config (tailwind.config.js)
‎
‎```js
‎module.exports = {
‎  content: ['./app/**/*.{js,jsx,ts,tsx}', './components/**/*.{js,jsx,ts,tsx}'],
‎  theme: {
‎    extend: {},
‎  },
‎  plugins: [],
‎}
‎```
‎
‎---
‎
‎## Key files (full code below)
‎
‎> The following sections include the important source files. You can copy each file into your project at the listed path.
‎
‎---
‎
‎### lib/supabaseClient.js
‎
‎```js
‎// lib/supabaseClient.js
‎import { createClient } from '@supabase/supabase-js';
‎
‎const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL;
‎const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY;
‎
‎export const supabase = createClient(supabaseUrl, supabaseAnonKey);
‎```
‎
‎---
‎
‎### lib/evaluation.js
‎
‎```js
‎// lib/evaluation.js
‎// Deterministic evaluation helpers (used by server-side endpoint and optionally client-side).
‎
‎import levenshtein from 'fast-levenshtein';
‎
‎export function normalizeText(t) {
‎  return (t || '').toString().trim().toLowerCase();
‎}
‎
‎export function fuzzyScoreShortAnswer(answerText, expectedKeywords = [], maxPoints = 1) {
‎  // expectedKeywords: array of keywords or phrases with optional weights: [{k:'map', w:1}, {k:'survey', w:0.5}]
‎  const provided = normalizeText(answerText);
‎  if (!provided) return 0;
‎
‎  let totalWeight = 0;
‎  let score = 0;
‎
‎  expectedKeywords.forEach((kw) => {
‎    const k = typeof kw === 'string' ? kw : kw.k;
‎    const w = typeof kw === 'string' ? 1 : (kw.w || 1);
‎    totalWeight += w;
‎    const nl = normalizeText(k);
‎    if (provided.includes(nl)) {
‎      score += w; // exact substring match
‎    } else {
‎      // fuzzy check via Levenshtein (small allowance)
‎      const dist = levenshtein.get(nl, provided.slice(0, nl.length + 5));
‎      if (dist <= Math.floor(nl.length * 0.3)) score += w * 0.8;
‎    }
‎  });
‎
‎  if (totalWeight === 0) return 0;
‎  const ratio = score / totalWeight;
‎  return Math.round(ratio * maxPoints * 100) / 100; // two decimals
‎}
‎
‎export function evaluateAttempt(questions = [], answers = {}) {
‎  // questions: array of {id, type, answer, points}
‎  // answers: {questionId: userAnswer}
‎  let totalPoints = 0;
‎  let obtained = 0;
‎  const perQuestion = [];
‎
‎  questions.forEach((q) => {
‎    totalPoints += q.points || 1;
‎    const userAns = answers[q.id];
‎    let qScore = 0;
‎    if (q.type === 'mcq') {
‎      const correct = q.answer; // choice id or array
‎      if (Array.isArray(correct)) {
‎        // multiple correct
‎        const matched = (userAns || []).filter((a) => correct.includes(a)).length;
‎        qScore = Math.round(((matched / correct.length) * (q.points || 1)) * 100) / 100;
‎      } else {
‎        qScore = userAns === correct ? (q.points || 1) : 0;
‎      }
‎    } else if (q.type === 'short') {
‎      // answer expected keywords
‎      const keywords = q.answer && q.answer.keywords ? q.answer.keywords : [];
‎      qScore = fuzzyScoreShortAnswer(userAns || '', keywords, q.points || 1);
‎    }
‎
‎    obtained += qScore;
‎    perQuestion.push({ id: q.id, score: qScore, max: q.points || 1 });
‎  });
‎
‎  const percent = totalPoints === 0 ? 0 : Math.round((obtained / totalPoints) * 10000) / 100;
‎  return {
‎    totalPoints,
‎    obtained,
‎    percent,
‎    perQuestion,
‎    strengths: deriveStrengths(perQuestion, questions),
‎    improvements: deriveImprovements(perQuestion, questions),
‎  };
‎}
‎
‎function deriveStrengths(perQuestion, questions) {
‎  // naive grouping: if user scored >=80% on a question, mark that topic as strength
‎  // When tests have tagging metadata, use it; here we fallback to question text keyword.
‎  const strengths = [];
‎  perQuestion.forEach((p) => {
‎    const q = questions.find((x) => x.id === p.id);
‎    if (!q) return;
‎    const ratio = p.max === 0 ? 0 : p.score / p.max;
‎    if (ratio >= 0.8) strengths.push({ id: q.id, hint: q.text.slice(0, 120) });
‎  });
‎  return strengths.slice(0, 5);
‎}
‎
‎function deriveImprovements(perQuestion, questions) {
‎  const items = [];
‎  perQuestion.forEach((p) => {
‎    const q = questions.find((x) => x.id === p.id);
‎    if (!q) return;
‎    const ratio = p.max === 0 ? 0 : p.score / p.max;
‎    if (ratio < 0.6) items.push({ id: q.id, hint: q.text.slice(0, 140) });
‎  });
‎  return items.slice(0, 7);
‎}
‎```
‎
‎> Note: This file uses `fast-levenshtein`. If you prefer not to add that package, replace with a small Levenshtein helper.
‎
‎---
‎
‎### pages/api/evaluate.js (serverless endpoint)
‎
‎```js
‎// pages/api/evaluate.js
‎import { supabase } from '../../lib/supabaseClient';
‎import { evaluateAttempt } from '../../lib/evaluation';
‎
‎export default async function handler(req, res) {
‎  if (req.method !== 'POST') return res.status(405).end();
‎
‎  const { userId, testId, answers } = req.body;
‎  if (!userId || !testId || !answers) return res.status(400).json({ error: 'Missing fields' });
‎
‎  // Fetch test questions from supabase
‎  const { data: questions, error } = await supabase
‎    .from('questions')
‎    .select('*')
‎    .eq('test_id', testId);
‎
‎  if (error) return res.status(500).json({ error: error.message });
‎
‎  const evalResult = evaluateAttempt(questions, answers);
‎
‎  // Save attempt
‎  const insert = await supabase.from('attempts').insert([
‎    {
‎      user_id: userId,
‎      test_id: testId,
‎      finished_at: new Date().toISOString(),
‎      score: evalResult.percent,
‎      details: { perQuestion: evalResult.perQuestion }
‎    }
‎  ]);
‎
‎  if (insert.error) {
‎    console.error('save attempt failed', insert.error);
‎  }
‎
‎  return res.status(200).json({ result: evalResult });
‎}
‎```
‎
‎---
‎
‎### pages/api/admin/tests.js (basic admin CRUD)
‎
‎```js
‎// pages/api/admin/tests.js
‎import { supabase } from '../../../lib/supabaseClient';
‎
‎export default async function handler(req, res) {
‎  // This endpoint expects server-side role to be set if you want full privileges.
‎  // You might protect with RLS using user JWTs; simplified here.
‎  const { method } = req;
‎  if (method === 'GET') {
‎    const { data, error } = await supabase.from('tests').select('*');
‎    if (error) return res.status(500).json({ error: error.message });
‎    return res.status(200).json({ data });
‎  }
‎
‎  if (method === 'POST') {
‎    const payload = req.body;
‎    const { data, error } = await supabase.from('tests').insert([payload]);
‎    if (error) return res.status(500).json({ error: error.message });
‎    return res.status(201).json({ data });
‎  }
‎
‎  return res.status(405).end();
‎}
‎```
‎
‎---
‎
‎### app/layout.jsx (global layout)
‎
‎```jsx
‎// app/layout.jsx
‎import '../styles/globals.css';
‎import { supabase } from '../lib/supabaseClient';
‎import Navbar from '../components/Navbar';
‎
‎export default function RootLayout({ children }) {
‎  return (
‎    <html lang="en">
‎      <body className="min-h-screen bg-slate-50 text-slate-800">
‎        <Navbar />
‎        <main className="max-w-4xl mx-auto p-4">{children}</main>
‎      </body>
‎    </html>
‎  );
‎}
‎```
‎
‎---
‎
‎### components/Navbar.jsx
‎
‎```jsx
‎import Link from 'next/link';
‎
‎export default function Navbar() {
‎  return (
‎    <nav className="bg-white shadow-sm">
‎      <div className="max-w-4xl mx-auto px-4 py-3 flex items-center justify-between">
‎        <Link href="/" className="text-xl font-semibold">{process.env.NEXT_PUBLIC_APP_NAME || 'SkillInsight'}</Link>
‎        <div className="space-x-3">
‎          <Link href="/auth/login" className="text-sm">Log in</Link>
‎          <Link href="/auth/register" className="text-sm font-medium">Register</Link>
‎          <Link href="/dashboard" className="text-sm">Dashboard</Link>
‎        </div>
‎      </div>
‎    </nav>
‎  );
‎}
‎```
‎
‎---
‎
‎### app/page.jsx (landing)
‎
‎```jsx
‎import Link from 'next/link';
‎
‎export default function Home() {
‎  return (
‎    <div className="py-12">
‎      <h1 className="text-3xl font-bold">Welcome to SkillInsight</h1>
‎      <p className="mt-4 text-slate-600">Take skill-based assessments, get Intelligent Feedback, and download your certificate.</p>
‎      <div className="mt-6 flex gap-3">
‎        <Link href="/auth/register" className="btn">Get Started</Link>
‎        <Link href="/dashboard" className="btn-outlined">Browse Tests</Link>
‎      </div>
‎    </div>
‎  );
‎}
‎```
‎
‎---
‎
‎### app/auth/register/page.jsx (register flow)
‎
‎```jsx
‎'use client'
‎import { useState } from 'react';
‎import { supabase } from '../../../lib/supabaseClient';
‎import { useRouter } from 'next/navigation';
‎
‎export default function Register() {
‎  const [email, setEmail] = useState('');
‎  const [name, setName] = useState('');
‎  const [password, setPassword] = useState('');
‎  const router = useRouter();
‎
‎  async function onRegister(e) {
‎    e.preventDefault();
‎    const { data, error } = await supabase.auth.signUp({ email, password }, { data: { full_name: name } });
‎    if (error) return alert(error.message);
‎    // Create profile
‎    await supabase.from('profiles').upsert({ id: data.user.id, full_name: name });
‎    router.push('/dashboard');
‎  }
‎
‎  return (
‎    <div className="max-w-md mx-auto mt-8 bg-white p-6 rounded-lg shadow-sm">
‎      <h2 className="text-xl font-semibold">Create account</h2>
‎      <form onSubmit={onRegister} className="mt-4 space-y-3">
‎        <input required value={name} onChange={(e)=>setName(e.target.value)} placeholder="Full name" className="w-full p-2 border rounded" />
‎        <input required type="email" value={email} onChange={(e)=>setEmail(e.target.value)} placeholder="Email" className="w-full p-2 border rounded" />
‎        <input required type="password" value={password} onChange={(e)=>setPassword(e.target.value)} placeholder="Password" className="w-full p-2 border rounded" />
‎        <button className="w-full py-2 bg-sky-600 text-white rounded">Register</button>
‎      </form>
‎    </div>
‎  );
‎}
‎```
‎
‎---
‎
‎### app/auth/login/page.jsx
‎
‎```jsx
‎'use client'
‎import { useState } from 'react';
‎import { supabase } from '../../../lib/supabaseClient';
‎import { useRouter } from 'next/navigation';
‎
‎export default function Login() {
‎  const [email, setEmail] = useState('');
‎  const [password, setPassword] = useState('');
‎  const router = useRouter();
‎
‎  async function onLogin(e) {
‎    e.preventDefault();
‎    const { error } = await supabase.auth.signInWithPassword({ email, password });
‎    if (error) return alert(error.message);
‎    router.push('/dashboard');
‎  }
‎
‎  return (
‎    <div className="max-w-md mx-auto mt-8 bg-white p-6 rounded-lg shadow-sm">
‎      <h2 className="text-xl font-semibold">Log in</h2>
‎      <form onSubmit={onLogin} className="mt-4 space-y-3">
‎        <input required type="email" value={email} onChange={(e)=>setEmail(e.target.value)} placeholder="Email" className="w-full p-2 border rounded" />
‎        <input required type="password" value={password} onChange={(e)=>setPassword(e.target.value)} placeholder="Password" className="w-full p-2 border rounded" />
‎        <button className="w-full py-2 bg-sky-600 text-white rounded">Log in</button>
‎      </form>
‎    </div>
‎  );
‎}
‎```
‎
‎---
‎
‎### components/QuestionRenderer.jsx
‎
‎```jsx
‎// components/QuestionRenderer.jsx
‎'use client'
‎import { useState } from 'react';
‎
‎export default function QuestionRenderer({ question, onAnswer }) {
‎  const [value, setValue] = useState(question.type === 'mcq' ? null : '');
‎
‎  function handleChange(v) {
‎    setValue(v);
‎    onAnswer(question.id, v);
‎  }
‎
‎  if (question.type === 'mcq') {
‎    return (
‎      <div className="p-3 bg-white rounded shadow-sm">
‎        <p className="font-medium">{question.text}</p>
‎        <div className="mt-2 space-y-2">
‎          {question.choices && question.choices.map((c) => (
‎            <label key={c.id} className="flex items-center gap-2">
‎              <input type="radio" name={question.id} checked={value===c.id} onChange={()=>handleChange(c.id)} />
‎              <span>{c.text}</span>
‎            </label>
‎          ))}
‎        </div>
‎      </div>
‎    );
‎  }
‎
‎  return (
‎    <div className="p-3 bg-white rounded shadow-sm">
‎      <p className="font-medium">{question.text}</p>
‎      <textarea value={value} onChange={(e)=>handleChange(e.target.value)} className="mt-2 w-full p-2 border rounded" rows={4}></textarea>
‎    </div>
‎  );
‎}
‎```
‎
‎---
‎
‎### app/dashboard/page.jsx (list tests)
‎
‎```jsx
‎'use client'
‎import { useEffect, useState } from 'react';
‎import Link from 'next/link';
‎import { supabase } from '../../lib/supabaseClient';
‎
‎export default function Dashboard() {
‎  const [tests, setTests] = useState([]);
‎
‎  useEffect(()=>{
‎    (async ()=>{
‎      const { data } = await supabase.from('tests').select('*');
‎      setTests(data || []);
‎    })();
‎  },[]);
‎
‎  return (
‎    <div className="space-y-4">
‎      <h2 className="text-2xl font-semibold">Available Skill Tests</h2>
‎      <div className="grid gap-3">
‎        {tests.map(t => (
‎          <div key={t.id} className="p-4 bg-white rounded shadow-sm flex justify-between items-center">
‎            <div>
‎              <div className="font-medium">{t.title}</div>
‎              <div className="text-sm text-slate-500">{t.description}</div>
‎            </div>
‎            <Link href={`/dashboard/test/${t.id}`} className="px-3 py-1 bg-sky-600 text-white rounded">Start</Link>
‎          </div>
‎        ))}
‎      </div>
‎    </div>
‎  );
‎}
‎```
‎
‎---
‎
‎### app/dashboard/test/[id]/page.jsx (test runner)
‎
‎```jsx
‎'use client'
‎import { useEffect, useState } from 'react';
‎import { useRouter, useSearchParams } from 'next/navigation';
‎import QuestionRenderer from '../../../../components/QuestionRenderer';
‎import { supabase } from '../../../../lib/supabaseClient';
‎
‎export default function TestRunner({ params }) {
‎  const { id } = params;
‎  const [questions, setQuestions] = useState([]);
‎  const [answers, setAnswers] = useState({});
‎  const [user, setUser] = useState(null);
‎  const [submitting, setSubmitting] = useState(false);
‎  const router = useRouter();
‎
‎  useEffect(()=>{
‎    (async ()=>{
‎      const s = await supabase.auth.getUser();
‎      setUser(s.data?.user || null);
‎      const { data } = await supabase.from('questions').select('*').eq('test_id', id);
‎      setQuestions(data || []);
‎    })();
‎  }, [id]);
‎
‎  function handleAnswer(qid, val) {
‎    setAnswers(prev => ({ ...prev, [qid]: val }));
‎  }
‎
‎  async function submit() {
‎    setSubmitting(true);
‎    const res = await fetch('/api/evaluate', {
‎      method: 'POST',
‎      headers: { 'Content-Type': 'application/json' },
‎      body: JSON.stringify({ userId: user.id, testId: id, answers }),
‎    });
‎    const data = await res.json();
‎    if (res.ok) {
‎      router.push(`/dashboard/results/${data.attemptId || 'na'}`);
‎    } else {
‎      alert('Evaluation failed');
‎    }
‎    setSubmitting(false);
‎  }
‎
‎  return (
‎    <div className="space-y-4">
‎      <h2 className="text-2xl font-semibold">Test</h2>
‎      <div className="space-y-3">
‎        {questions.map(q => (
‎          <QuestionRenderer key={q.id} question={q} onAnswe
