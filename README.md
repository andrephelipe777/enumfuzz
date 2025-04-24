usage: trabalho2.py [-h] [-u USER_AGENT]
domain subdomain_wordlist path_wordlist output_file

Exemplo: python3 trabalho2.py site.com.br subdominioslista.txt listadefuzzing.txt nomedoarquivo.txt

#!/usr/bin/python3
import argparse
import requests

def fuzz_subdomains(domain, wordlist, user_agent):
    subdomains = []
    headers = {'User-Agent': user_agent}
    
    with open(wordlist, 'r') as file:
        for line in file:
            subdomain = line.strip() + '.' + domain
            url = f"http://{subdomain}"
            try:
                response = requests.get(url, headers=headers)
                if response.status_code == 200:
                    subdomains.append(subdomain)
                    print(f"[+] Encontrado: {subdomain}")
            except requests.ConnectionError:
                pass
            except requests.exceptions.InvalidURL:
                print(f"[!] URL inválida: {url}")
    
    return subdomains

def fuzz_paths(domain, wordlist, user_agent):
    paths = []
    headers = {'User-Agent': user_agent}
    
    with open(wordlist, 'r') as file:
        for line in file:
            path = line.strip()
            url = f"http://{domain}/{path}"
            try:
                response = requests.get(url, headers=headers)
                if response.status_code == 200:
                    paths.append(url)
                    print(f"[+] Encontrado: {url}")
            except requests.ConnectionError:
                pass
            except requests.exceptions.InvalidURL:
                print(f"[!] URL inválida: {url}")
    
    return paths

def save_results(results, output_file):
    with open(output_file, 'w') as file:
        for result in results:
            file.write(result + '\n')

def main():
    parser = argparse.ArgumentParser(description="Fuzzing e enumeração de subdomínios e caminhos")
    parser.add_argument("domain", help="Domínio alvo")
    parser.add_argument("subdomain_wordlist", help="Caminho para a wordlist de subdomínios")
    parser.add_argument("path_wordlist", help="Caminho para a wordlist de caminhos")
    parser.add_argument("output_file", help="Arquivo para salvar os resultados")
    parser.add_argument("-u", "--user-agent", default="Mozilla/5.0", help="User agent a ser utilizado")

    args = parser.parse_args()
    
    try:
        subdomains = fuzz_subdomains(args.domain, args.subdomain_wordlist, args.user_agent)
        paths = fuzz_paths(args.domain, args.path_wordlist, args.user_agent)
        results = subdomains + paths
    except KeyboardInterrupt:
        print("\n[!] Interrupção detectada. Salvando resultados parciais...")
    finally:
        save_results(results, args.output_file)
        print(f"[+] Resultados salvos em {args.output_file}")

if __name__ == "__main__":
    main()




