"""Command-line interface.

Examples:
  python main.py demo
  python main.py bfs Alice
  python main.py dfs Alice
  python main.py mutual Alice Eve
  python main.py path Alice Judy
  python main.py recommend Alice
  python main.py components
  python main.py benchmark --sizes 1000 10000 100000
  python main.py --data my_edges.csv bfs Alice
"""
import argparse
import os

from social_network import (Graph, bfs, dfs, mutual_connections, shortest_path,
                            recommend_friends, connected_components)
from social_network.benchmark import run_benchmark, save_csv, save_plot

DEFAULT_DATA = os.path.join("data", "sample_network.csv")


def demo(g):
    print(f"Loaded {g}\n")
    print("BFS from Alice :", " -> ".join(bfs(g, "Alice")))
    print("DFS from Alice :", " -> ".join(dfs(g, "Alice")))
    print("Mutual (Alice, Eve):", sorted(mutual_connections(g, "Alice", "Eve")))
    p = shortest_path(g, "Alice", "Judy")
    print("Shortest Alice -> Judy:", " -> ".join(p), f"({len(p) - 1} hops)")
    print("Recommendations for Alice:", recommend_friends(g, "Alice"))
    comps = connected_components(g)
    print(f"Connected components: {len(comps)} (sizes {[len(c) for c in comps]})")


def main():
    ap = argparse.ArgumentParser(description="Social network analysis")
    ap.add_argument("--data", default=DEFAULT_DATA, help="edge list CSV (user_a,user_b)")
    sub = ap.add_subparsers(dest="cmd", required=True)
    sub.add_parser("demo")
    for name in ("bfs", "dfs", "recommend"):
        sp = sub.add_parser(name); sp.add_argument("user")
    for name in ("mutual", "path"):
        sp = sub.add_parser(name); sp.add_argument("a"); sp.add_argument("b")
    sub.add_parser("components")
    bp = sub.add_parser("benchmark")
    bp.add_argument("--sizes", type=int, nargs="+", default=[1000, 5000, 10000, 50000, 100000])
    bp.add_argument("--degree", type=int, default=10)
    bp.add_argument("--repeats", type=int, default=5)
    bp.add_argument("--kind", choices=["random", "hub"], default="random")
    bp.add_argument("--out", default="results")
    args = ap.parse_args()

    if args.cmd == "benchmark":
        rows = run_benchmark(args.sizes, args.degree, args.repeats, args.kind)
        os.makedirs(args.out, exist_ok=True)
        save_csv(rows, os.path.join(args.out, f"benchmark_{args.kind}.csv"))
        save_plot(rows, os.path.join(args.out, f"benchmark_{args.kind}.png"))
        print(f"\nSaved CSV and chart in '{args.out}/'")
        return

    g = Graph.from_csv(args.data)
    try:
        if args.cmd == "demo":
            demo(g)
        elif args.cmd == "bfs":
            print(" -> ".join(bfs(g, args.user)))
        elif args.cmd == "dfs":
            print(" -> ".join(dfs(g, args.user)))
        elif args.cmd == "mutual":
            print(sorted(mutual_connections(g, args.a, args.b)))
        elif args.cmd == "path":
            p = shortest_path(g, args.a, args.b)
            print("No connection found." if p is None
                  else " -> ".join(p) + f"  ({len(p) - 1} hops)")
        elif args.cmd == "recommend":
            for who, n in recommend_friends(g, args.user):
                print(f"{who}: {n} mutual friend(s)")
        elif args.cmd == "components":
            for i, c in enumerate(connected_components(g), 1):
                print(f"Component {i} ({len(c)} users): {sorted(c)}")
    except KeyError as e:
        print("Error:", e)


if __name__ == "__main__":
    main()
