# Python 从DataV.GeoAtlas中抽取省、市、区县中心经纬度数据

```python
import requests
import pandas as pd

BASE_URL = 'https://geo.datav.aliyun.com/areas_v3/bound/{}_full.json'


def get_feature_list(url):
    try:
        req = requests.get(url)
        req.raise_for_status()
        return req.json()['features']
    except Exception as e:
        print(f"Error fetching data from {url}: {e}")
        return None


if __name__ == '__main__':

    url = 'https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json'
    res = []
    pattern = '\d+'
    data = get_feature_list(url)
    if not data:
        print("Failed to fetch province data.")
        exit()
    for province in data:
        print(province)
        province_code = province['properties']['adcode']
        if province_code == '100000_JD':
            continue
        province_name = province['properties']['name']
        province_lon, province_lat = province['properties']['center']
        print(province_code, province_name)

        url = BASE_URL.format(province_code)
        province_data = get_feature_list(url)
        if province_data is None:
            res.append([province_name, province_code, province_lon, province_lat,
                        '', '', '', '',
                        '', '', '', ''])
            continue

        for city in province_data:
            city_code = city['properties']['adcode']
            city_name = city['properties']['name']
            city_lon, city_lat = city['properties']['center']
            print(city_code, city_name)
            url = BASE_URL.format(city_code)
            district_data = get_feature_list(url)
            if district_data is None:
                res.append([province_name, province_code, province_lon, province_lat
                               , city_name, city_code, city_lon, city_lat,
                            '', '', '', ''])
                continue

            for district in district_data:
                district_code = district['properties']['adcode']
                district_name = district['properties']['name']
                lon, lat = district['properties']['center']

                res.append([province_name, province_code, province_lon, province_lat
                               , city_name, city_code, city_lon, city_lat
                               , district_name, district_code, lon, lat])
    result = pd.DataFrame(res, columns=['省区划名称', '省区划代码', '省经度', '省维度',
                                    '市区划名称', '市区划代码', '市经度', '市维度',
                                    '区县区划名称', '区县区划代码', '区县经度', '区县维度'])
    result.to_excel(r'D:\data\data\data.xlsx', index=False)

    print(result)

```

![image-20240923224849924](https://docs-pics.oss-cn-chengdu.aliyuncs.com/images/202409232248132.png)