# Parse the log file 
import re 
import matplotlib.pyplot as plt
import matplotlib
matplotlib.use('TkAgg')

def parse_log(log_file):
        log_line_re = re.compile(r'^(?P<timestamp>\d{4}/\d{2}/\d{2} \d{2}:\d{2}:\d{2}) \[(?P<ip>[^\]]+)\] (?P<message>.*)')
        with open(log_file, 'r', encoding='utf-8') as file:
         for line in file:
           m = log_line_re.match(line.strip())
           if m:
               entry=m.groupdict()
               print(f"Timestamp: {entry['timestamp']}, IP: {entry['ip']}, Message: {entry['message']}")
           else:
               print(f"Unrecognized line format: {line.strip()}")

def generate_report(log_file):
    api_re=re.compile(r'^(?P<method>GET|POST|PUT|DELETE) (?P<path>/\S+) (?P<status>\d{3}) (?P<duration>[\d\.]+)µs')
    id_re=re.compile(r'\[(\d{4}[A-Z0-9]+)\]')
    total_requests = 0
    status_codes = {}
    endpoint_counts = {}
    endpoint_items = {}
    unique_ids=set()
    year_counts = {}
    total_timetables= 0
    backtracking_count = 0
    iterative_random_sampling_count = 0
    with open(log_file, 'r', encoding='utf-8') as file:
        for line in file:
           m = log_line_re.match(line.strip())           
           if m:
                message=m.group('message')
                if "Backtracking" in message:
                    backtracking_count += 1
                if "Iterative Random Sampling" in message:
                    iterative_random_sampling_count += 1
                api_match=api_re.match(message)
                if api_match:
                    total_requests += 1
                    status=api_match.group('status')
                    status_codes[status] = status_codes.get(status, 0)+1
                    path=api_match.group('path')
                    endpoint_counts[path] = endpoint_counts.get(path,0)+1
                    duration=float(api_match.group('duration'))
                    if path not in endpoint_items:
                        endpoint_items[path] = []
                    endpoint_items[path].append(duration)
                    if path == '/courses':
                        total_timetables += 1
                id_match = id_re.search(message)
                if id_match:
                    sid=id_match.group(1)
                    unique_ids.add(sid)
                    year = sid[:4]
                    year_counts[year] = year_counts.get(year, 0) + 1  
    if len(unique_ids)>0:
        avg_timetables = total_timetables/len(unique_ids)
    else:
        avg_timetables = 0                   
    print("------------------------\nTraffic and Usage Anlytics\n------------------------")
    print(f"Total API requests: {total_requests}\n")
    print("HTTP Status codes:")
    for scode, count in status_codes.items():
        print(f"{scode}: {count} requests {count/total_requests*100:.2f}%")
    print("\nEndpoint Popularity:")
    for endpoint,count in endpoint_counts.items():
        print(f"{endpoint}:{count}")
    print("-------------------------\nPerformance Metrics\n------------------------")
    for endpoint, times in endpoint_items.items():
        avg_time= sum(times)/len(times)
        max_time = max(times)
        print(f"{endpoint}: \n  -Average = {avg_time:.2f}µs,\n  -Max = {max_time:.2f}µs")
    print("------------------------\n Unique ID analysis\n------------------------") 
    print(f"Unique IDs visited:{len(unique_ids)}\n")
    print("Yearly ID counts:")
    for year, count in year_counts.items():
        print(f"{year}: {count}")
    print("------------------------\nApplication-Specific Insights\n------------------------")
    print(f"Total timetables generated: {total_timetables}")
    print(f"\nAverage timetables generated per user: {avg_timetables}")
    print(f"  -Backtracking algorithm used: {backtracking_count} times")
    print(f"  -Iterative Random Sampling algorithm used: {iterative_random_sampling_count} times")
    visualize_data(endpoint_counts, endpoint_items, year_counts)
# Visualize data
def visualize_data(endpoint_counts, endpoint_items, year_counts):
    plt.style.use('dark_background')
    plt.figure(figsize=(8,5))
    endpoints = list(endpoint_counts.keys())
    count = list(endpoint_counts.values())
    plt.bar(endpoints, count, color='cyan')
    plt.title('Endpoint Popularity', color='white')
    plt.xlabel('Endpoints', color='white')
    plt.ylabel('Number of Requests', color='white')
    plt.xticks(rotation=45, color='white')
    plt.yticks(color='white')
    plt.tight_layout()
    plt.show()
    endpoints_graph = list(endpoint_items.keys())
    avg_times = [sum(times)/len(times) for times in endpoint_items.values()]
    plt.figure(figsize=(8,5))
    plt.bar(endpoints_graph, avg_times, color='magenta')
    plt.title('Average Response Time per Endpoint', color='white')
    plt.xlabel('Endpoints', color='white')
    plt.ylabel('Average Response Time (µs)', color='white')
    plt.xticks(rotation=45, color='white')
    plt.yticks(color='white')
    plt.tight_layout()
    plt.show()
    years=list(year_counts.keys())
    counts=list(year_counts.values())
    plt.figure(figsize=(8,5))
    plt.bar(years, counts, color='green')
    plt.title('Unique ID Counts for every Year', color='white')
    plt.xlabel('Year', color='white')   
    plt.ylabel('Number of Unique IDs', color='white')
    plt.xticks(rotation=45, color='white')
    plt.yticks(color='white')
    plt.tight_layout()
    plt.show()


if __name__ == "__main__":
    parse_log('timetable.log')
    generate_report('timetable.log')
    
 
