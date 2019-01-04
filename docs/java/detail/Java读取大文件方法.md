```java
public static List<Long> parser(File sourcFile, Long spuId) throws FileNotFoundException, IOException {

        List<Long> offerIdList = new ArrayList<Long>();
        String remain = null;

        FileChannel channel = new RandomAccessFile(sourcFile, "r").getChannel();
        ByteBuffer buffer = ByteBuffer.allocate(BUFFER_SIZE);
        byte[] dst = new byte[BUFFER_SIZE];
        int size;
        while ((size = channel.read(buffer)) != -1) {
            buffer.rewind();
            buffer.get(dst);
            // 解析数据
            String data = new String(dst);
            remain = buildOfferIdList(data.trim(), remain, spuId, offerIdList);
            buffer.clear();
        }

        // 处理剩余的数据
        if (remain != null && !"".equals(remain)) {
            String[] endStrings = remain.split(LEFT_SEP);
            if (endStrings.length > 1 && Long.parseLong(endStrings[1].trim()) == spuId) offerIdList.add(Long.parseLong(endStrings[0].trim()));
        }
        return offerIdList;
    }


public static String buildOfferIdList(String data, String remain, Long spuId, List<Long> offerIdList) {
        if (data != null && !"".equals(data)) {
            if (remain != null && !"".equals(remain)) {
                data = remain + data.trim();
            }
            int index = data.lastIndexOf(RIGHT_SEP);
            if (index > 0) {
                String aa = data.substring(0, index);
                remain = data.substring(index + 1);
                String[] bb = aa.split(RIGHT_SEP);
                for (int i = 0; i < bb.length; i++) {
                    String cc = bb[i];
                    String[] dd = cc.split(LEFT_SEP);
                    if (dd.length > 1 && Long.parseLong(dd[1].trim()) == spuId) {
                        offerIdList.add(Long.parseLong(dd[0].trim()));
                    } else if (dd.length <= 1) { // 对脏数据打日志
                        if (log.isWarnEnabled()) {
                            log.warn("Dirty data: " + cc);
                        }
                    }
                }
            } else {
                return data;
            }
            return remain;
        }
        return remain;
    }



 public static String buildOfferSpuIdsList(String data, String remain, Map<Long, Long> offerspuids) {
        if (data != null && !"".equals(data)) {
            if (remain != null && !"".equals(remain)) {
                data = remain + data;
            }
            int index = data.lastIndexOf(RIGHT_SEP);
            if (index > 0) {
                String aa = StringUtil.substring(data, 0, index);
                remain = StringUtil.substring(data, index + 1);
                String[] bb = StringUtil.split(aa, RIGHT_SEP);
                String cc = null;
                String[] dd = null;
                for (int i = 0; i < bb.length; i++) {
                    cc = StringUtil.trim(bb[i]);
                    dd = cc.split(LEFT_SEP);
                    if (dd.length > 1 && !StringUtil.isEmpty((dd[0])) && !StringUtil.isEmpty((dd[1]))) {
                        offerspuids.put(Long.parseLong(dd[0]), Long.parseLong(dd[1]));
                    } else { // 对脏数据打日志
                        if (log.isWarnEnabled()) {
                            log.warn("Dirty data: " + cc);
                        }
                    }

                }

            } else {
                return data;
            }
            return remain;
        }
        return remain;
    }public static void parser(File file, long start, long size, Map<Long, Long> offerSpuIds) throws Exception {
        String remain = null;
        RandomAccessFile raf = null;
        FileChannel channel = null;
        MappedByteBuffer inputBuffer = null;
        String data = null;
        try {

            raf = new RandomAccessFile(file, "r");
            channel = raf.getChannel();
            inputBuffer = channel.map(FileChannel.MapMode.READ_ONLY, start, size);

            byte[] dst = new byte[BUFFER_SIZE];

            data = null;

            for (int offset = 0; offset < inputBuffer.capacity(); offset += BUFFER_SIZE) {
                if (inputBuffer.capacity() - offset >= BUFFER_SIZE) {
                    for (int i = 0; i < BUFFER_SIZE; i++) {
                        dst[i] = inputBuffer.get(offset + i);
                    }
                } else {
                    for (int i = 0; i < inputBuffer.capacity() - offset; i++) {
                        dst[i] = inputBuffer.get(offset + i);
                    }
                }

                data = new String(dst);
                remain = buildOfferSpuIdsList(data, remain, offerSpuIds);

            }

            // 处理剩余的数据
            if (remain != null && !"".equals(remain)) {
                String[] end = remain.split(LEFT_SEP);
                if (end.length > 1 && !StringUtil.isEmpty((end[0])) && !StringUtil.isEmpty((end[1]))) {
                    offerSpuIds.put(Long.parseLong(end[0]), Long.parseLong(end[1]));
                }
            }

        } finally {
            // for gc
            inputBuffer.clear();
            inputBuffer = null;
            channel.close();
            channel = null;
            raf.close();
            raf = null;
            data = null;
        }

    }
```