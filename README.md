import matplotlib.pyplot as plt
import networkx as nx
from matplotlib.animation import FuncAnimation

def maximal_product_graph(G, H):
    product_graph = nx.DiGraph()
    for u in G.nodes():
        for v in H.nodes():
            product_graph.add_node((u, v))
    for u1, u2 in G.edges():
        for v1, v2 in H.edges():
            product_graph.add_edge((u1, v1), (u2, v2))
    return product_graph

def burn_graph(G, sources, max_steps=10):
    burned = set(sources)
    new_burned = set(sources)
    history = [set(burned)]
    
    for _ in range(max_steps):
        current_burned = set(new_burned)
        new_burned = set()
        for node in current_burned:
            for neighbor in G.neighbors(node):
                if neighbor not in burned:
                    new_burned.add(neighbor)
                    burned.add(neighbor)
        history.append(set(burned))
        if len(burned) == len(G.nodes()):
            break
            
    return history

def aborescence_number(G):
    root = next(iter(G.nodes))
    directed_tree = nx.bfs_tree(G, root)
    return directed_tree

# Define Graphs
G = nx.DiGraph()
G.add_edges_from([
    ('c1', 'c2'), ('c2', 'c3'), ('c3', 'c1'),
    ('c1', 'c4'), ('c4', 'c5'), ('c5', 'c2')
])

H = nx.DiGraph()
H.add_edges_from([
    ('p1', 'p2'), ('p2', 'p3'), ('p3', 'p4'),
    ('p1', 'p5'), ('p5', 'p4')
])

# Create Maximal Product Graph
product_graph = maximal_product_graph(G, H)

# Generate consistent layout **after** creating product graph
pos = nx.spring_layout(product_graph, k=1.5, iterations=50)

# Compute Properties
dominating_set = nx.dominating_set(product_graph)
aborescence_tree = aborescence_number(product_graph)
sources = [list(product_graph.nodes())[0]]
history = burn_graph(product_graph, sources)

# Plot Results
fig, ax = plt.subplots(1, 4, figsize=(20, 5))

# Maximal Product Graph
ax[0].set_title("Maximal Product Graph")
nx.draw(product_graph, pos, with_labels=True, node_size=800, node_color='lightgrey', ax=ax[0])

# Maximal Product Graph with Domination Set
ax[1].set_title("Maximal Product Graph with Domination Set")
node_colors = ['lightblue' if node not in dominating_set else 'orange' for node in product_graph.nodes()]
nx.draw(product_graph, pos, with_labels=True, node_color=node_colors, node_size=800, ax=ax[1])

# Highlight Dominating Nodes
for node in dominating_set:
    nx.draw_networkx_nodes(product_graph, pos, nodelist=[node], node_color='orange', node_size=1000, ax=ax[1])

# Aborescence Tree
ax[2].set_title("Aborescence Tree")
nx.draw(aborescence_tree, pos, with_labels=True, node_size=800, node_color='lightgreen', edge_color='blue', ax=ax[2])

# Burning Number Animation (Fixed)
def update(step):
    ax[3].clear()
    ax[3].set_title(f"Burning Number (Maximal) - Step {step + 1}")
    
    # Draw original product graph
    nx.draw(product_graph, pos, ax=ax[3], node_color='lightgrey', with_labels=True, node_size=800)
    
    # Highlight burned nodes correctly
    if step < len(history):
        nx.draw_networkx_nodes(product_graph, pos, nodelist=list(history[step]), node_color='red', node_size=800, ax=ax[3])

ani = FuncAnimation(fig, update, frames=len(history), repeat=False, interval=500)

plt.tight_layout()
plt.show()
