```java
public void init() {

        try {
            //URL url = Thread.currentThread().getContextClassLoader().getResource("data/clearedCatIdList");
            //File file = new File(this.getClass().getResource("/data/clearedCatIdList").toURI());
            //File file = new File(url.getPath());
            InputStream in=ClassLoader.getSystemResourceAsStream("data/clearedCatIdList");
            BufferedReader reader = null;
            reader = new BufferedReader(new InputStreamReader(in));
            String text = null;
            while ((text = reader.readLine()) != null) {
                CLEARED_CATEGORY_IDS.add(Long.parseLong(text.trim()));
            }
            System.out.println(CLEARED_CATEGORY_IDS.toString());
        } catch (Exception e) {
            if (logger.isErrorEnabled()) {
                logger.error("init catIdList error: " + e.getMessage());
            }
        }
    }
```