# Directus with Next.js Integration Guide

## 1. Introduction

Directus is an open-source **headless CMS** that wraps your database with a real-time API and an intuitive admin app.  
This guide explains how to connect **Directus** with **Next.js**, run it locally, and integrate it into your Supabase or standalone backend infrastructure.

---

## 2. Prerequisites

You need:
- Node.js and npm installed.
- Docker (optional, for containerized setup).
- Access to a running PostgreSQL or Supabase database.
- A working Next.js project.

---

## 3. Setup Directus Locally

You can create a Directus project locally using **NPX**:

```bash
npx create-directus-project my-directus-app
cd my-directus-app
```

Update the `.env` file with your database credentials:

```env
DB_CLIENT=pg
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=mydb
DB_USER=postgres
DB_PASSWORD=password
PORT=8055
```

Start the Directus server:

```bash
npx directus start
```

Access the admin panel at [http://localhost:8055](http://localhost:8055) and complete the setup wizard.

---

## 4. Dockerize the Directus Application

You can easily run Directus using Docker for consistency across environments.

Create a `docker-compose.yml` file:

```yaml
version: "3"
services:
  directus:
    image: directus/directus:latest
    ports:
      - "8055:8055"
    environment:
      KEY: value
    depends_on:
      - db
  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: example
      POSTGRES_DB: directus
```

Run the application:

```bash
docker-compose up -d
```

---

## 5. Connect Directus with Next.js

You can query your Directus CMS from a Next.js app using its REST or GraphQL API.

### Example (REST API)

```tsx
// app/page.tsx
async function getArticles() {
  const res = await fetch(`${process.env.NEXT_PUBLIC_DIRECTUS_URL}/items/articles`);
  return res.json();
}

export default async function Home() {
  const { data } = await getArticles();
  return (
    <div>
      <h1>Articles</h1>
      <ul>
        {data.map((item: any) => (
          <li key={item.id}>{item.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Example (GraphQL API)

```tsx
const query = `
query {
  articles {
    id
    title
    description
  }
}`;

const res = await fetch(`${process.env.NEXT_PUBLIC_DIRECTUS_URL}/graphql`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ query })
});
const data = await res.json();
```

This allows Next.js to fetch CMS data dynamically from Directus.

---

## 6. Integration with Supabase

You can host Directus on the same database used by Supabase since both support PostgreSQL.  
To do so, point your Directus `.env` to your Supabase database credentials.

Example:

```env
DB_CLIENT=pg
DB_HOST=db.supabase.co
DB_PORT=5432
DB_DATABASE=postgres
DB_USER=postgres
DB_PASSWORD=<your-supabase-password>
```

This ensures unified access between Directus CMS and Supabase-managed backend.

---

## 7. Deploying Directus with GCP

- **Frontend (Next.js):** Deploy on GCP.
- **Backend (Directus):** Deploy via Docker container on a **GCP VM** instance.

Once deployed, update your environment file in Next.js:

```env
NEXT_PUBLIC_DIRECTUS_URL=https://your-directus-domain.com
```

---

## 8. Next Steps

- Configure **authentication** via Directus roles and permissions.
- Enable **webhooks** for real-time sync with Supabase.
- Use **Directus SDK** for simplified API calls in Next.js:
  ```bash
  npm install @directus/sdk
  ```
- Explore the [Directus documentation](https://docs.directus.io) for advanced setup.

---

## 9. Conclusion

You now have a fully functional setup where **Directus** acts as a CMS and API layer, and **Next.js** handles the frontend.  
This architecture provides flexibility, scalability, and clean content management for your applications.
