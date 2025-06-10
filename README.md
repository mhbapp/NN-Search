negative_keywords = DEFAULT_NEGATIVE_KEYWORDS + [kw.strip() for kw in custom_kw.split(",") if kw.strip()]
entities = [n.strip() for n in names_input.splitlines() if n.strip()]

newsapi = NewsApiClient(api_key=api_key)
aggregated_rows: List[Dict] = []

progress = st.progress(0)
for idx, entity in enumerate(entities, start=1):
    with st.spinner(f"Searching for negative news about: {entity}"):
        articles = search_entity(entity, newsapi, negative_keywords)
        for art in articles:
            aggregated_rows.append(
                {
                    "Entity": entity,
                    "Title": art.get("title"),
                    "Source": art.get("source", {}).get("name"),
                    "Published": art.get("publishedAt")[:10] if art.get("publishedAt") else None,
                    "URL": art.get("url"),
                }
            )
    progress.progress(idx / len(entities))

if aggregated_rows:
    df = pd.DataFrame(aggregated_rows)
    st.subheader("⚠️ Negative Matches Found")
    st.dataframe(df, use_container_width=True)
    csv_bytes = df.to_csv(index=False).encode("utf-8")
    st.download_button("Download CSV report", data=csv_bytes, file_name="negative_news_report.csv", mime="text/csv")
else:
    st.success("🎉 No negative news found for any entries in the specified time window.")
