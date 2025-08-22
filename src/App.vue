<script setup lang="ts">
import { ref, onMounted } from 'vue';

const stars = ref(0);
const commits = ref(0);
const forks = ref(0);
const followers = ref(0);

const accounts = ["ApplePieCodes", "AlulaOneshot"];
const accountStats = ref<{ username: string; stars: number; commits: number; forks: number; followers: number }[]>([]);

// 🔑 optional token support
const headers: HeadersInit = {};
if (import.meta.env.VITE_GITHUB_TOKEN) {
  headers.Authorization = `Bearer ${import.meta.env.VITE_GITHUB_TOKEN}`;
}

async function fetchGitHubData() {
  try {
    let totalStars = 0;
    let totalCommits = 0;
    let totalForks = 0;
    let totalFollowers = 0;

    const results = await Promise.all(
      accounts.map(async (username) => {
        let userStars = 0;
        let userCommits = 0;
        let userForks = 0;
        let userFollowers = 0;

        // --- User profile ---
        const userRes = await fetch(`https://api.github.com/users/${username}`, { headers });
        const userData = await userRes.json();
        userFollowers = userData.followers ?? 0;

        // --- User repos ---
        const reposRes = await fetch(`https://api.github.com/users/${username}/repos?per_page=100`, { headers });
        const repos = await reposRes.json();

        userStars = repos.reduce((sum: number, repo: any) => sum + (repo.stargazers_count ?? 0), 0);
        userForks = repos.reduce((sum: number, repo: any) => sum + (repo.forks_count ?? 0), 0);

        const commitCounts = await Promise.all(
          repos
            .filter((r: any) => !r.fork)
            .map(async (repo: any) => {
              try {
                const commitsRes = await fetch(`https://api.github.com/repos/${username}/${repo.name}/contributors`, { headers });
                const contributors = await commitsRes.json();
                const user = contributors.find((c: any) => c.login === username);
                return user ? user.contributions : 0;
              } catch {
                return 0;
              }
            })
        );
        userCommits = commitCounts.reduce((a, b) => a + b, 0);

        // --- Add to totals ---
        totalStars += userStars;
        totalCommits += userCommits;
        totalForks += userForks;
        totalFollowers += userFollowers;

        // --- Push per-account stats ---
        return { username, stars: userStars, commits: userCommits, forks: userForks, followers: userFollowers };
      })
    );

    // Update reactive totals + per-account stats
    stars.value = totalStars;
    commits.value = totalCommits;
    forks.value = totalForks;
    followers.value = totalFollowers;
    accountStats.value = results;
  } catch (e) {
    console.error("GitHub data fetch failed:", e);
  }
}

onMounted(fetchGitHubData);
</script>

<template>
  <div class="absolute top-0 left-0 min-h-screen min-w-screen bg-gray-900">
    <div class="py-2 my-10 mx-10 rounded-lg bg-gradient-to-r from-indigo-500 via-purple-500 to-pink-500">
      <h1 class="ml-6 mt-1 text-6xl ubuntu-mono-regular">Hello. I'm Liam.</h1>
      <h2 class="ml-6 mt-1 text-2xl ubuntu-mono-regular">I'm a student at CodeRVA and highly skilled in many areas of Computer Science.</h2>
      <h2 class="ml-6 mt-1 text-2xl ubuntu-mono-regular">I spend my time rebuilding modern technologies you work with every day</h2>

      <img class="ml-6 my-4"
        src="https://skillicons.dev/icons?i=arch,atom,bash,c,cs,cpp,clion,codepen,css,debian,deno,dotnet,firebase,git,github,godot,haxe,html,js,linux,lua,md,mint,pycharm,py,raspberrypi,rider,robloxstudio,rust,supabase,svelte,swift,tailwind,ts,ubuntu,visualstudio,vscode,vue,webstorm,windows&perline=10"
        alt="Skill Icons"/>

      <!-- Combined totals -->
      <div class="mx-8 grid grid-cols-4 gap-10 text-center text-xl ubuntu-mono-regular">
        <p>{{ stars }} Stars</p>
        <p>{{ commits }} Commits</p>
        <p>{{ forks }} Forks</p>
        <p>{{ followers }} Followers</p>
      </div>

      <!-- Per-account breakdown -->
      <div class="mt-10">
        <h3 class="text-center text-2xl ubuntu-mono-regular">Per-Account Stats</h3>
        <div class="mt-4 grid grid-cols-1 mx-6 md:grid-cols-2 gap-6 text-center">
          <div v-for="acc in accountStats" :key="acc.username" class="p-4 bg-gray-800 rounded-xl shadow-lg">
            <a :href="`https://github.com/${acc.username}`" target="_blank" class="text-xl font-bold text-indigo-300">
              {{ acc.username }}
            </a>
            <p class="text-white">{{ acc.stars }} Stars</p>
            <p class="text-white">{{ acc.commits }} Commits</p>
            <p class="text-white">{{ acc.forks }} Forks</p>
            <p class="text-white">{{ acc.followers }} Followers</p>
          </div>
        </div>
      </div>
    </div>
    <div class="grid grid-cols-2 gap-6">
      <div class="py-2 px-2 mb-10 ml-10 rounded-lg bg-gradient-to-r from-indigo-500 via-purple-500 to-pink-500 block">
        <h1 class="text-3xl text-black ubuntu-mono-regular">About Me</h1>
        <p class="text-1xl text-black ubuntu-mono-regular">
            My first introduction to programming was in 2017. I was in 1st grade and my friend showed me a website called "Scratch". I played some games and was fascinated by the idea of creating games so easily. I looked at the 'code' and everything made sense. The blockes all clicked (pun intended). Everything worked together to achive an outcome. Piece by piece, the computer followed the instructions given to it by the user. I made several projects with scratch and eventually I could program in scratch with ease. I still remember perfectly how to make something in scratch to this day. In 2018, the beta of scratch 3.0 released. I was excited to have so many new blocks to play with. As the years went by, I found myself falling further into programming. I began trying to make actual apps for desktop computers. I messed around with python, but found it too complicated to work with. <i>2021</i>. I was bored. I couldn't find anything to do. I decided to mess around in python again. This time though, something was different. A new tool called ChatGPT released. Curious, I created an account. Working with ChatGPT, I created a (barebones and glitchy) code editor in QT5. I was ecstatic. Programming had never been so easy! I kept learning, this time the correct way. Older and more experienced in the internet, I tried to learn more advanced programming with python. In 2023, I discovered a tool called SoloLearn. It was (and is) an online classroom for programming, similar to duolingo. I spent the entire year working on sololearn. I learned C#, JavaScript, HTML, and many more languages. It was the biggest jump in knowledge I had experienced. The year after, I learned Rust. It was at this time that I began to learn low level development. I created my first attempt at an operating system (lithium). Later, I creted the kobold kernel. It barely did anything, but it was pretty cool to me. After that, I created my modern youtube channel (as opposed to my really old ones, they aren't relevant). I posted my first video (<a href="https://youtu.be/sDJ68mENp_I?si=SpRsXXUeVQ7TwYd4">Warning, Swear Word</a>) and for some reason it got 1300 views. I never replicated that success. And now you're cought up to today. I'm a student at CodeRVA, starting my first year.
        </p>
      </div>
    </div>
  </div>
</template>
