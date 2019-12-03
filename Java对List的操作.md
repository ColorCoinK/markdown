---
title: 去除 List<Object> 中的重复数据
toc: true
layout: blog
categories:
  - blog
  - Java
tags:
  - Java
  - List
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->
# 从`List<Object>` 中取相同列，转为另一个`list`	

```java
private static List<UserUrl> initData() {
	List<UserUrl> list = new ArrayList<>();

	UserUrl url = new UserUrl("20181107153500", "1", 120);
	list.add(url);
	url = new UserUrl("20181107153500", "2", 120);
	list.add(url);
	url = new UserUrl("20181107153500", "3", 156);
	list.add(url);
	url = new UserUrl("20181107153500", "4", 120);
	list.add(url);
	url = new UserUrl("20181107153500", "5", 156);
	list.add(url);
	url = new UserUrl("20181107153500", "6", 120);
	list.add(url);

	url = new UserUrl();

	url = new UserUrl("20181107153000", "1", 120);
	list.add(url);
	url = new UserUrl("20181107153000", "2", 156);
	list.add(url);
	url = new UserUrl("20181107153000", "3", 120);
	list.add(url);
	url = new UserUrl("20181107153000", "4", 156);
	list.add(url);
	url = new UserUrl("20181107153000", "5", 156);
	list.add(url);
	url = new UserUrl("20181107153000", "6", 120);
	list.add(url);

	// 校验时间格式
	String reg = "(\\d{4})(\\d{2})(\\d{2})(\\d{2})(\\d{2})(\\d{2})";
	List<UserUrl> result = new ArrayList<>();
	for (UserUrl userUrl : list) {
		String time = userUrl.getProcess_time();
		// 更改时间格式
		time = time.replaceAll(reg, "$2-$3 $4:$5");
		userUrl.setProcess_time(time);
		result.add(userUrl);
	}
	
	System.err.println(result);
	return result;
}

/**
 * 
 * @Title: queryActiveTrend
 * @Description: 活跃用户趋势
 *  	应用Map中键唯一的特性，将相同统计时间节点的数据合并为一个object
 * @param @return
 * @return Object
 * @throws
 */
public static List<ActiveTrendVO> queryActiveTrend() {
	// 1.得到查询结果:process_time,type,user_num
	List<UserUrl> list = initData();// userCountDao.queryActiveUserTrend();
	System.out.println(list.toString());
	
	// 2.将统计时间相同的数据封装为 ActiveTrendVO
	List<ActiveTrendVO> result = new ArrayList<>();
	/**
	/// 嵌套的 list
	List<ActiveUserVO> nestvo = new ArrayList<>();

	Map<String, Object> map = new HashMap<>();

	/// 第一条数据结果缺失,采用第二种方案
	for (UserUrl userUrl : list) {
		if (map.containsKey(userUrl.getProcess_time())) {
			ActiveUserVO user = new ActiveUserVO(userUrl.getType(), userUrl.getUser_num());
			nestvo.add(user);
		} else {
			ActiveTrendVO trend = new ActiveTrendVO(userUrl.getProcess_time(), nestvo);
			result.add(trend);
			nestvo.clear();
		}
		map.put(userUrl.getProcess_time(), userUrl.getProcess_time());
	}
	 */
	TreeMap<String, List<UserUrl>> tm = new TreeMap<>();
	for (int i = 0; i < list.size(); i++) {
		UserUrl userUrl = list.get(i);
		String time = userUrl.getProcess_time();
		if (tm.containsKey(time)) {
			ArrayList<UserUrl> tempList = (ArrayList<UserUrl>) tm.get(time);
			tempList.add(userUrl);
		} else {
			ArrayList<UserUrl> temlist = new ArrayList<>();
			temlist.add(userUrl);
			tm.put(time, temlist);
		}
	}

	for (String key : tm.keySet()) {
		List<UserUrl> userList = tm.get(key);
		ArrayList<ActiveUserVO> nestList = new ArrayList<>();
		for (UserUrl userUrl : userList) {
			ActiveUserVO active = new ActiveUserVO(userUrl.getType(), userUrl.getUser_num());
			nestList.add(active);
		}
		ActiveTrendVO trend = new ActiveTrendVO(key, nestList);
		result.add(trend);
	}
	
	return result;
}


public static void main(String[] args) {
	List<UserUrl> list = initData();

	// 2.将统计时间相同的数据封装为 ActiveTrendVO
	List<ActiveTrendVO> result = new ArrayList<>();
	/// 嵌套的 list
	TreeMap<String, List<UserUrl>> tm = new TreeMap<>();
	for (int i = 0; i < list.size(); i++) {
		UserUrl userUrl = list.get(i);
		String time = userUrl.getProcess_time();
		if (tm.containsKey(time)) {
			ArrayList<UserUrl> tempList = (ArrayList<UserUrl>) tm.get(time);
			tempList.add(userUrl);
		} else {
			ArrayList<UserUrl> temlist = new ArrayList<>();
			temlist.add(userUrl);
			tm.put(time, temlist);
		}
	}

	for (String key : tm.keySet()) {
		List<UserUrl> userList = tm.get(key);
		ArrayList<ActiveUserVO> nestList = new ArrayList<>();
		for (UserUrl userUrl : userList) {
			ActiveUserVO active = new ActiveUserVO(userUrl.getType(), userUrl.getUser_num());
			nestList.add(active);
		}
		ActiveTrendVO trend = new ActiveTrendVO(key, nestList);
		result.add(trend);
	}
	System.out.println(JSON.toJSONString(result));
}
```

```java
private static List<UserAccountStatus> initData() {
	List<UserAccountStatus> list = new ArrayList<>();

	UserAccountStatus userAccountStatus = new UserAccountStatus(4822, "1", "四月");
	list.add(userAccountStatus);
	userAccountStatus = new UserAccountStatus(352, "2", "四月");
	list.add(userAccountStatus);
	UserAccountStatus userAccountStatus1 = new UserAccountStatus(44, "1", "五月");
	list.add(userAccountStatus1);
	userAccountStatus1 = new UserAccountStatus(0, "2", "五月");
	list.add(userAccountStatus1);
	UserAccountStatus userAccountStatus2 = new UserAccountStatus(406, "1", "六月");
	list.add(userAccountStatus2);
	userAccountStatus2 = new UserAccountStatus(27, "2", "六月");
	list.add(userAccountStatus2);
	UserAccountStatus userAccountStatus3 = new UserAccountStatus(15, "1", "七月");
	list.add(userAccountStatus3);
	userAccountStatus3 = new UserAccountStatus(0, "2", "七月");
	list.add(userAccountStatus3);
	UserAccountStatus userAccountStatus4 = new UserAccountStatus(0, "1", "八月");
	list.add(userAccountStatus4);
	userAccountStatus4 = new UserAccountStatus(0, "2", "八月");
	list.add(userAccountStatus4);
	UserAccountStatus userAccountStatus5 = new UserAccountStatus(1, "1", "九月");
	list.add(userAccountStatus5);
	userAccountStatus5 = new UserAccountStatus(0, "2", "九月");
	list.add(userAccountStatus5);
	UserAccountStatus userAccountStatus6 = new UserAccountStatus(511, "1", "十月");
	list.add(userAccountStatus6);
	userAccountStatus6 = new UserAccountStatus(6, "2", "十月");
	list.add(userAccountStatus6);
	UserAccountStatus userAccountStatus7 = new UserAccountStatus(15, "1", "十一月");
	list.add(userAccountStatus7);
	userAccountStatus7 = new UserAccountStatus(1, "2", "十一月");
	list.add(userAccountStatus7);

	System.err.println(JSON.toJSONString(list));
	return list;
}

/**
 * 
 * @Title: queryOpenAccountGrowth
 * @Description: 开户用户历史增长情况
 * @return
 * Object
 */
public List<ActiveTrendVO> queryOpenAccountGrowth() {
	// 1.查询数据库得按 月、类型 分组的用户统计数
	List<UserAccountStatus> userAccountStatusList = initData(）//userCountDao.queryUserAccuontStatusGroupByMonth();
	List<ActiveTrendVO> result = new ArrayList<>();

	LinkedHashMap<String, List<UserAccountStatus>> linkMap = new LinkedHashMap<>();

	// 2.使用 process_time 作为键
	for (UserAccountStatus userAccountStatus : userAccountStatusList) {
		String time = userAccountStatus.getProcessTime();
		if (linkMap.containsKey(time)) {
			ArrayList<UserAccountStatus> tempList = (ArrayList<UserAccountStatus>) linkMap.get(time);
			tempList.add(userAccountStatus);
		} else {
			ArrayList<UserAccountStatus> temList = new ArrayList<>();
			temList.add(userAccountStatus);
			linkMap.put(time, temList);
		}
	}
	// 3.遍历key,转换实体类
	for (String key : linkMap.keySet()) {
		System.err.println("process_time:" + key);
		List<UserAccountStatus> list = linkMap.get(key);
		ArrayList<ActiveUserVO> nestList = new ArrayList<>();

		for (UserAccountStatus userAccountStatus : list) {
			ActiveUserVO activeUserVO = new ActiveUserVO(userAccountStatus.getType(),
					userAccountStatus.getUserNum());
			nestList.add(activeUserVO);
		}
		ActiveTrendVO trend = new ActiveTrendVO(key, nestList);
		result.add(trend);
	}
	return result;
}

```

# 对连续数据取最后一条作为结果显示 

```java
private List<UserUrl> formartPlayTrackResult(List<UserUrl> list) {
	// 数组下标
	int index = 0;
	for (UserUrl userUrl : list) {
		userUrl.setIndex(index++);
	}

	List<UserUrl> result = new ArrayList<>(new HashSet<>(list));

	// set去重后恢复排序
	UserUrl[] temp = new UserUrl[result.size()];
	result.toArray(temp);

	Arrays.sort(temp, new Comparator<UserUrl>() {
		@Override
		public int compare(UserUrl userUrl1, UserUrl userUrl2) {
			return userUrl1.getIndex() - userUrl2.getIndex();
		}
	});

	result = Arrays.asList(temp);
	result = new ArrayList<>(result);
	SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm");
	try {
		for (int i = 0; i < result.size(); i++) {
			for (int j = i + 1; j < result.size();) {
				// 第一条播放记录的播放时间,示例值： 2018-11-13 14:05
				String firstTime = result.get(i).getProcess_time();

				// 第二条播放记录播放时间
				String secondTime = result.get(j).getProcess_time();

				String firstName = result.get(i).getChannel_name();
				String secondName = result.get(j).getChannel_name();

				// 两条记录的时间间隔
				long difference = sdf.parse(secondTime).getTime() - sdf.parse(firstTime).getTime();
				difference = difference / (1000 * 60);

				// 时间间隔为5分钟
				if (difference == 5 && firstName.equals(secondName)) {
					result.remove(i);
				} else {
					j++;
				}

			}
		}
	} catch (Exception e) {
		// TODO: handle exception
	}
	return result;
}
```