# web 前端
## boostrap
* boostrap 表格
  * ```js
     // angular.forEach($scope.data.config_list,function (item) {
                        //     console.log(JSON.stringify(item,null,4))
                        // }) 
    ```
   ```js
      //		DefaultMultipartHttpServletRequest defaultMultipartHttpServletRequest = (DefaultMultipartHttpServletRequest)inv.getRequest();
  //		Map map = defaultMultipartHttpServletRequest.getParameterMap();
  //		Map <String,MultipartFile> mapfile = defaultMultipartHttpServletRequest.getFileMap();
  //		System.out.println(map.keySet());
  //		System.out.println(mapfile.keySet());
  //
  //		for(String key : mapfile.keySet()){
  //			System.out.println(mapfile.get(key).getOriginalFilename());
  //		}
  //		String [] values = (String[])map.get("productconfig");
  //		for(int i=0;i<values.length;i++){
  //			System.out.println(values[i]);
  //		}
  //		System.out.println("*************************8");
  //		System.out.println(defaultMultipartHttpServletRequest.getParameter("productconfig"));
  //		System.out.println(defaultMultipartHttpServletRequest.getFile("listiconfile").getOriginalFilename());
  //		System.out.println(defaultMultipartHttpServletRequest.getFile("detailiconfile").getOriginalFilename());
 ```
 	@Post("/createConfig2")
	@HttpFeatures(charset = "utf-8")
	public String addCondfig(Invocation inv ) throws IOException {
		ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
		IOUtils.copyLarge(inv.getRequest().getInputStream(),byteArrayOutputStream);
		String data = new String(byteArrayOutputStream.toByteArray(), Charsets.UTF_8);
		JSONObject pc = null;
		try {
			 pc = new JSONObject(data);
			logger.info("update_broker_list_data:{}", pc.toString());
			System.out.println(data);
			productConfigLists.add(pc);

			JSONObject ret =new JSONObject();
			ret.put("code",0);
			ret.put("data",pc);
			return  "@"+ret.toString();
		} catch (JSONException e) {
			logger.error("hit an error {}",e);
			return "{\"code\":-1,\"msg\":\"Hit an impossible json error!!\"}";
		}
	}
```

```java


package com.xiaomi.xiaoqiang.task;

import com.xiaomi.miliao.thrift.ClientFactory;
import com.xiaomi.miliao.zookeeper.ZKFacade;
import com.xiaomi.xiaoqiang.common.utils.EmailUtils;
import com.xiaomi.xiaoqiang.common.utils.VersionConverter;
import com.xiaomi.xiaoqiang.common.utils.XQConstants;
import com.xiaomi.xiaoqiang.common.utils.basic.XQJsonUtils;
import com.xiaomi.xiaoqiang.common.utils.grayupgradev3.GrayUpgradeFilterType;
import com.xiaomi.xiaoqiang.common.utils.grayupgradev3.GrayUpgradePlatform;
import com.xiaomi.xiaoqiang.service.thrift.gen.GrayUpgradeV3;
import com.xiaomi.xiaoqiang.service.thrift.gen.ConfigItem;
import com.xiaomi.xiaoqiang.task.wrsst.HDFSUtils;
import org.apache.http.NameValuePair;
import com.xiaomi.xiaoqiang.common.utils.DataStatsHelper;
import net.sf.json.JSON;
import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.LocatedFileStatus;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.fs.RemoteIterator;
import org.apache.thrift.TException;
import org.elasticsearch.search.aggregations.metrics.percentiles.InternalPercentiles;
import org.json.JSONObject;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import com.xiaomi.xiaoqiang.service.thrift.gen.XiaoQiangStandaloneService;
import scala.util.parsing.combinator.testing.Str;

import java.net.URI;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.Iterator;
import java.util.List;

public class DeviceRomChannelCountStat {
    private static Logger logger = LoggerFactory.getLogger(DeviceActivationLocationStat.class);

    public static final String clusterAZOR = "azorprc-xiaomi";
    private static final XiaoQiangStandaloneService.Iface xiaoqiangService
            = ClientFactory.getClient(XiaoQiangStandaloneService.Iface.class, 10000);
    private static GrayUpgradePlatform platform = GrayUpgradePlatform.X04G;
    private static JSONObject versionCount = new JSONObject();
    private static JSONObject versionChannelClosed = new JSONObject();
    private static JSONObject versionChannelOpen = new JSONObject();

    private static final String DirRouterDeviceStatistics = "/user/h_data_platform/platform/xiaoqiang/router_device_statistics/date=%04d%02d%02d";

    private static JSONObject clusterVersionChannelCount = new JSONObject();
    private static List<String> clusterList = new ArrayList<>();

    private static JSONObject clusterMappingJO = new JSONObject();
    public static void main(String[] args) {
        clusterMappingJO.put(DataStatsHelper.clusterZJY,"CN Cluster");
        clusterMappingJO.put(DataStatsHelper.clusterAZDE,"EU Cluster");
        clusterMappingJO.put(DataStatsHelper.clusterAZMB,"IN Cluster");
        clusterMappingJO.put(DataStatsHelper.clusterKSMOS,"RU Cluster");
        clusterMappingJO.put(DataStatsHelper.clusterALSG,"SG Cluster");
        clusterMappingJO.put(clusterAZOR,"US Cluster");
        try {
            Calendar caYesterday = Calendar.getInstance();
            caYesterday.add(Calendar.DATE,-1);
            String inputPath = HDFSUtils.makeDateString(DirRouterDeviceStatistics, caYesterday);
            logger.info("raw data path = : {}", inputPath);
            for (NameValuePair nvp : DataStatsHelper.clusterList) {
                String cluster = nvp.getName();
                if(cluster.equals(DataStatsHelper.clusterZJY) || cluster.equals(DataStatsHelper.clusterKSMOS))
                {
                    continue;
                }
                clusterList.add(cluster);
            }
            clusterList.add(clusterAZOR);
            getChannel();
            for(String currentCluster : clusterList) {
                loadAndAnalysis(currentCluster, inputPath);

                logger.info("rom-count {}", versionCount.toString());
                logger.info("versionChannelClosed {}", versionChannelClosed.toString());
                logger.info("versionChannelOPen{}", versionChannelOpen.toString());
                JSONObject versionChannelCount = new JSONObject();

                Iterator it = versionCount.keys();
                while (it.hasNext()) {
                    String version = (String) it.next();
                    String channel;
                    if (versionChannelOpen.has(version)) {
                        channel = versionChannelOpen.optString(version);
                    } else {
                        channel = versionChannelClosed.optString(version, "N/A");
                    }
                    JSONObject channelCount = new JSONObject();
                    channelCount.put("count", versionCount.optString(version));
                    channelCount.put("channel", channel);
                    versionChannelCount.put(version, channelCount);
                }

                clusterVersionChannelCount.put(clusterMappingJO.optString(currentCluster,"N/A"), versionChannelCount);
            }//clusterList
            logger.debug("clusterVersionChannelCount: {}", clusterVersionChannelCount.toString());
            sendEmail(clusterVersionChannelCount);
            logger.debug("send email ok");
        }catch (Exception e){
            logger.error("error {}",e);
        }
    }
    private static void loadAndAnalysis(String cluster, String inputPath) throws Exception {
        inputPath = String.format("hdfs://%s%s", cluster, inputPath);
        logger.info("cluster path {}",inputPath);
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(URI.create(inputPath), conf);
        RemoteIterator<LocatedFileStatus> files = fs.listFiles(new Path(inputPath), true);
        while(files.hasNext()){
            LocatedFileStatus file = files.next();
            if(file.isFile() && file.getLen() > 0){
                String filepath = file.getPath().toString();
                HDFSUtils.hdfsReadFile(fs, filepath, new HDFSUtils.LineCallback() {
                    @Override
                    public void handleLine(String line) {
                        String[] splits = StringUtils.splitPreserveAllTokens(line, "\t");
                        if (splits.length < 5) {
                            logger.error("invalidLine {} ", line);
                            return;
                        }
                        String source = splits[0];
                        String hardware = splits[1];
                        String romVersion = splits[2];
                        String statType = splits[3];
                        String count = splits[4];
                        if(!hardware.equalsIgnoreCase(platform.getName()))
                        {
                            return;
                        }
//                        logger.debug("stat: source:{},hardware:{},rom:{},statType:{},count:{}", source, hardware, romVersion, statType, count);
                        versionCount.put(romVersion,count);
                    }
                    @Override
                    public void complete() {
                    }
                });
            }
        }

    }
    private static void getChannel() throws TException {
        ConfigItem configItem = xiaoqiangService.getConfigItem(
                XQConstants.ConfigName.DATA_NS, "mico/gray_upgrade_notify"
        );
        String invalidValue = "NA";
        if (StringUtils.isNotBlank(configItem.value)) {
            JSONObject configJO = XQJsonUtils.optParseJson(configItem.value);
            JSONObject hardwareConfigJO = configJO.optJSONObject(platform.getName());
            JSONObject channelMappingJO = hardwareConfigJO.optJSONObject("channelMapping");
            JSONObject wlMappingJO = hardwareConfigJO.optJSONObject("wlMapping");
//            clusterMappingJO = configJO.optJSONObject("clusterMapping");
            if (channelMappingJO == null || wlMappingJO == null) {
                logger.debug("channelMapping/wlMapping empty!");
                return;
            }
            List<GrayUpgradeV3> upgradeList = xiaoqiangService.getGrayUpgradeV3List(platform.getCode(), 10, 0, 1000);
            for (GrayUpgradeV3 u : upgradeList) {
                int whiteListType = u.filterType == 0 ? u.filterType : GrayUpgradeFilterType.getMappingWhiteListValueType(u.filterType).getCode();
                if(!channelMappingJO.has(u.channel) || !wlMappingJO.has(String.valueOf(whiteListType))) {
                    continue;
                }
                String version  = VersionConverter.toString(u.version);
                String olderChannel = versionChannelOpen.optString(version, invalidValue);
                if (!olderChannel.equals(invalidValue)) {
                    continue;
                }
                String channelMapping = channelMappingJO.getString(u.channel);
                String wlMapping = wlMappingJO.getString(String.valueOf(whiteListType));
                String channelDesc = StringUtils.isBlank(wlMapping) ? channelMapping : String.format("%s-%s", channelMapping, wlMapping);
                versionChannelOpen.put(version, channelDesc);

                }

            upgradeList = xiaoqiangService.getGrayUpgradeV3List(platform.getCode(), 0, 0, 1000);
            for (GrayUpgradeV3 u : upgradeList) {
                int whiteListType = u.filterType == 0 ? u.filterType : GrayUpgradeFilterType.getMappingWhiteListValueType(u.filterType).getCode();
                if(!channelMappingJO.has(u.channel) || !wlMappingJO.has(String.valueOf(whiteListType))) {
                    continue;
                }
                String version  = VersionConverter.toString(u.version);
                String olderChannel = versionChannelClosed.optString(version, invalidValue);
                if (!olderChannel.equals(invalidValue)) {
                    continue;
                }
                String channelMapping = channelMappingJO.getString(u.channel);
                String wlMapping = wlMappingJO.getString(String.valueOf(whiteListType));
                String channelDesc = StringUtils.isBlank(wlMapping) ? channelMapping : String.format("%s-%s", channelMapping, wlMapping);
                versionChannelClosed.put(version, channelDesc);
            }
        }
    }
    private static void sortByCount(JSONObject versionChannelCount){
        Iterator it = versionChannelCount.keys();
        while(it.hasNext()){
            String version = (String)it.next();
            JSONObject channelCount = versionChannelCount.optJSONObject(version);
            
        }

    }
    private static void sendEmail(JSONObject clusterVersionChannelCount){
        String html = "<h2>"+"Mi Smart Clock Daily Statistics"+"</h2>";
        Iterator it =clusterVersionChannelCount.keys();
        while(it.hasNext()){
            String cluster = (String)it.next();
            JSONObject versionChannelCount = clusterVersionChannelCount.optJSONObject(cluster);
            Iterator its = versionChannelCount.keys();

            html += "<br><h3>"+cluster+" Cluster"+"</h3>";
            html +=
                    "<table  border=\"1\" cellspacing=\"0\" cellpadding=\"0\">" +
                            "<thead>"+
                            "<tr style=\"background-color:#5B9BD5\">"+
                            "<th align='center' style='width:14%;text-align:center;'>"+"Product Model"+"</th>"+
                            "<th align='center' style='width:14%;text-align:center;'>"+"Rom Version"+"</th>"+
                            "<th align='center' style='width:14%;text-align:center;'>"+"Channel"+"</th>"+
                            "<th align='center' style='width:14%;text-align:center;'>"+"On-Line Quantity"+"</th>"+
                            "</tr>"+
                            "</thead>"+
                            "<tbody>";
            if(its.hasNext()) {
                while (its.hasNext()) {
                    String version = (String) its.next();
                    JSONObject channelCount = versionChannelCount.optJSONObject(version);
                    String channel = channelCount.optString("channel");
                    String count = channelCount.optString("count");

                    html +=
                            "<tr style=\"background-color:#BFBFBF\">" +
                                    "<td align='center' style='width:14%;text-align:center;'>" + "Mi Smart Clock" + "</td>" +
                                    "<td align='center' style='width:14%;text-align:center;'>" + version + "</td>" +
                                    "<td align='center' style='width:14%;text-align:center;'>" + channel + "</td>" +
                                    "<td align='center' style='width:14%;text-align:center;'>" + count + "</td>" +
                                    "</tr>";
                }
            }
            else{
                html +=
                        "<tr style=\"background-color:#BFBFBF\">"+
                                "<td align='center' style='width:14%;text-align:center;'>"+"Mi Smart Clock"+"</td>"+
                                "<td align='center' style='width:14%;text-align:center;'>"+""+"</td>"+
                                "<td align='center' style='width:14%;text-align:center;'>"+""+"</td>"+
                                "<td align='center' style='width:14%;text-align:center;'>"+""+"</td>"+
                                "</tr>";

            }
            html += "</tbody>"+
                    "</table>";
        }
        String title = "X04G Daily Active Summary";
        EmailUtils.sendEmailByToken("zhouqiqi@xiaomi.com", null,null, title, html, null, null);
        logger.debug("mail content  {}",html);
    }
}
```
 