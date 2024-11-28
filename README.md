### 프로그램의 주요 기능과 클래스별 구현 방식

---

### 1. **GUI 인터페이스**

#### **구현 클래스**: `ControlPanel`, `MapPanel`

1. **출발지, 목적지 선택 (드롭다운 메뉴)**:
   - **클래스**: `ControlPanel`
   - **역할**:
     - 사용자가 출발지와 목적지를 선택할 수 있는 `JComboBox` 제공.
     - "최단 경로 찾기" 버튼을 통해 탐색 요청.
   - **핵심 코드**:
     ```java
     JComboBox<String> startComboBox = new JComboBox<>();
     JComboBox<String> endComboBox = new JComboBox<>();
     JButton findPathButton = new JButton("최단 경로 찾기");

     findPathButton.addActionListener(e -> {
         String startName = (String) startComboBox.getSelectedItem();
         String endName = (String) endComboBox.getSelectedItem();

         Node startNode = nodes.stream().filter(n -> n.name.equals(startName)).findFirst().orElse(null);
         Node endNode = nodes.stream().filter(n -> n.name.equals(endName)).findFirst().orElse(null);

         mapPanel.findAndDisplayShortestPath(startNode, endNode);
     });
     ```

2. **노드 시각화 (지도에 원형으로 표시)**:
   - **클래스**: `MapPanel`
   - **역할**:
     - `Graphics2D`를 사용해 지도 위에 노드(원)를 그리며, 출발지와 목적지를 강조 표시.
   - **핵심 코드**:
     ```java
     for (Node node : nodes) {
         if (highlightedNodes.contains(node)) {
             g2.setColor(Color.YELLOW); // 출발지/목적지 강조
         } else {
             g2.setColor(new Color(220, 20, 60)); // 일반 노드 색상
         }
         g2.fillOval(node.x - 15, node.y - 15, 30, 30);
     }
     ```

3. **경로 시각화 (무당 경로와 보도 경로)**:
   - **클래스**: `MapPanel`
   - **역할**:
     - 최단 경로를 무당 경로(빨간색)와 보도 경로(파란색)로 구분하여 시각화.
   - **핵심 코드**:
     ```java
     for (PathSegment segment : shortestPath) {
         g2.setColor(segment.isMudang ? Color.RED : Color.BLUE);
         g2.setStroke(new BasicStroke(segment.isMudang ? 4 : 3));
         g2.drawLine(segment.from.x, segment.from.y, segment.to.x, segment.to.y);
     }
     ```

---

### 2. **줌 및 드래그로 지도 탐색**

#### **구현 클래스**: `MapPanel`

1. **줌 (Zoom)**:
   - **역할**:
     - 마우스 휠 이벤트를 통해 `zoomLevel` 값을 조정하고, 화면 중심을 확대/축소.
   - **핵심 코드**:
     ```java
     addMouseWheelListener(e -> {
         double oldZoomLevel = zoomLevel;
         double delta = 0.1;
         zoomLevel = e.getWheelRotation() < 0
             ? Math.min(zoomLevel + delta, MAX_ZOOM)
             : Math.max(zoomLevel - delta, MIN_ZOOM);

         double zoomFactor = zoomLevel / oldZoomLevel;
         offsetX = zoomFactor * (offsetX - e.getPoint().x) + e.getPoint().x;
         offsetY = zoomFactor * (offsetY - e.getPoint().y) + e.getPoint().y;

         repaint();
     });
     ```

2. **드래그 (Drag)**:
   - **역할**:
     - 마우스 드래그 이벤트로 `offsetX`, `offsetY` 값을 조정해 뷰포트를 이동.
   - **핵심 코드**:
     ```java
     addMouseMotionListener(new MouseMotionAdapter() {
         @Override
         public void mouseDragged(MouseEvent e) {
             int dx = e.getX() - lastDragPoint.x;
             int dy = e.getY() - lastDragPoint.y;
             offsetX += dx;
             offsetY += dy;
             lastDragPoint = e.getPoint();
             repaint();
         }
     });
     ```

---

### 3. **경로 및 이동 거리 상세 정보 GUI 출력**

#### **구현 클래스**: `ControlPanel`

1. **총 이동 거리 및 경로 정보 출력**:
   - **역할**:
     - `Dijkstra` 결과를 기반으로 총 이동 거리, 무당 경로 구간 수, 보도 경로 구간 수를 출력.
   - **핵심 코드**:
     ```java
     JLabel pathSummaryLabel = new JLabel();
     findPathButton.addActionListener(e -> {
         PathInfo pathInfo = PathInfo.findPathInfo(startNode, endNode, edges, mudangs);
         pathSummaryLabel.setText(String.format(
             "총 거리: %d, 무당 구간: %d, 보도 구간: %d",
             pathInfo.totalDistance, pathInfo.mudangCount, pathInfo.roadCount
         ));
     });
     ```

2. **경로 세부 정보 출력**:
   - **역할**:
     - 세부 경로 정보를 노드 간 경로와 각 구간의 거리로 출력.
   - **핵심 코드**:
     ```java
     String detailedInfo = pathInfo.getDetailedPathInfo(edges, mudangs);
     pathDetailsLabel.setText("<html>" + detailedInfo.replace(" → ", "<br> → ") + "</html>");
     ```

---

### 4. **지도 이미지 추가**

#### **구현 클래스**: `MapPanel`

1. **역할**:
   - 지도 이미지를 `backgroundImage`로 불러와 배경에 표시.
2. **핵심 코드**:
   ```java
   ImageIcon icon = new ImageIcon("navigation-main/map.png");
   backgroundImage = icon.getImage();

   g2.drawImage(backgroundImage, 0, 0, getWidth(), getHeight(), this);
   ```

---

### 클래스별 요약

| **클래스**         | **역할**                                                                 |
|---------------------|-------------------------------------------------------------------------|
| **`ControlPanel`**  | 출발지/목적지 선택, 최단 경로 탐색, 경로 요약 및 세부 정보 출력 관리.   |
| **`MapPanel`**      | 지도 렌더링, 노드/경로 시각화, 줌 및 드래그 구현.                     |
| **`Dijkstra`**      | 최단 경로 탐색 알고리즘.                                               |
| **`PathInfo`**      | 경로 세부 정보 분석 및 요약.                                           |

