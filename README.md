# PageCraftML - Graph Neural Network for Layout Prediction

This module implements a Graph Neural Network (GNN) system for predicting layout sizes and alignments across different device resolutions based on desktop layouts.

## Overview

The system processes JSON templates where:
- **Desktop** is the main version with complete item definitions
- **Other resolutions** (Mobile, Tablet, etc.) are expected to have empty arrays initially
- The GNN predicts appropriate sizes and alignments for other resolutions

## Architecture

### Workflow

1. **Data Cleaning**: Extract only size/alignment relevant properties from desktop items
   - Properties: `width`, `height`, `imageWidth`, `imageHeight`, `alignItems`, `justifyContent`, `flexDirection`, `alignContent`
   - Removes: colors, styles, content, filters, etc.

2. **Graph Construction**: Build a graph representation from desktop items
   - Nodes: represent items
   - Edges: represent parent-child relationships and spatial proximity

3. **GNN Processing**: Predict properties for target resolutions
   - Currently uses rule-based scaling (ready for trained model integration)

4. **Data Merging**: Merge predicted properties back with original desktop items
   - Restores all previously purged properties (colors, styles, content, etc.)
   - Only updates size/alignment properties

### Files

- `nn_server.py`: FastAPI server endpoint that processes layout data
- `gnn_model.py`: GNN model architecture and prediction functions
- `data_processor.py`: Data cleaning and merging utilities
- `requirements.txt`: Python dependencies

## Installation

```bash
pip install -r requirements.txt
```

## Usage

Start the server:

```bash
python nn_server.py
```

The server will run on `http://localhost:8000`

### API Endpoint

**POST `/process`**

Accepts JSON payload with `itemsByResolution` structure:

```json
{
  "itemsByResolution": {
    "Desktop (1920x1080)": [...],  // Full desktop items
    "Tablet (768x1024)": [],        // Empty - will be predicted
    "Mobile (375x667)": []          // Empty - will be predicted
  }
}
```

Returns:

```json
{
  "processedPayload": {
    "itemsByResolution": {
      "Desktop (1920x1080)": [...],  // Unchanged
      "Tablet (768x1024)": [...],     // Predicted sizes/alignments
      "Mobile (375x667)": [...]       // Predicted sizes/alignments
    }
  }
}
```

## Model Training (Future Work)

The GNN model architecture is ready for training. To train:

1. Prepare a dataset of desktop layouts and their corresponding mobile/tablet layouts
2. Train the model using the `LayoutGNN` class
3. Save the trained model weights
4. Update `nn_server.py` to load the trained model

Example training code structure:

```python
model = LayoutGNN()
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

# Training loop
for epoch in range(num_epochs):
    # Build graphs from desktop items
    # Predict for target resolutions
    # Compute loss
    # Backpropagate
    # Update weights

torch.save(model.state_dict(), 'model_checkpoint.pth')
```

## Current Implementation

The current implementation uses **rule-based scaling** as a baseline:
- Sizes are scaled proportionally based on resolution ratios
- Mobile layouts switch to column flex direction
- Alignment properties are adapted based on screen size

This provides a working system that can be enhanced with actual GNN predictions once a trained model is available.

