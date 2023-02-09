<template>
  <svg id="svg" />
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'graph',
  computed: {
    nodes() {
      let nodes = this.graphData.pages
      .map(page =>
      ({
        id: page.id,
        title: page.title,
        x: this.width * Math.random(),
        y: this.height * Math.random(),
        rx: this.byteLength(page.title) * 2,
        ry: 10,
        user: false
      }))

      const users = this.graphData.users.map(user => ({
        id: user.id,
        title: user.name,
        x: this.width * Math.random(),
        y: this.height * Math.random(),
        rx: this.byteLength(user.name),
        ry: 10,
        user: true
      }))
      nodes = nodes.concat(users)
      return nodes
    },
    edges() {
      const ids = new Set(this.nodes.map(node => node.id))
      const idm = new Map()
      this.nodes.forEach((node, index) => {idm[node.id] = index})
      let edges = this.graphData.links
        .filter(edge => ids.has(edge.from) && ids.has(edge.to))
        .map(edge =>
        ({
          source: idm[edge.from],
          target: idm[edge.to],
          l: Math.random() * 150
        }))

      const userPages = this.graphData.userPages
        .filter(up => ids.has(up.user) && ids.has(up.page))
        .map(up =>
        ({
          source: idm[up.user],
          target: idm[up.page],
          l: Math.random() * 300
        })) 
      edges = edges.concat(userPages)
      return edges
    }
  },
  data: () => ({
    graphData: [],
    projects: ['help-jp', 'comic-forum', 'icons'],
    project: '',
    width: 0,
    height: 0
  }),
  async mounted () {
    this.project = this.projects[0]
    this.width = document.querySelector('svg').clientWidth
    this.height = document.querySelector('svg').clientHeight
    await this.fetchData()
    await this.render()
  },
  methods: {
    async fetchData() {
      const res = await fetch(`https://sb-graph-kondoumh.netlify.app/${this.project}_graph.json`, {
        mode: 'cors'
      })
      this.graphData = await res.json()
    },
    async render() {
      d3.select('svg').selectAll('*').remove()

      const zoom = d3.zoom()
        .scaleExtent([1/3, 40])
        .on('zoom', zoomed)

      d3.select('svg')
        .attr('viewBox', '0 0 1200 1400')
        .attr("preserveAspectRatio", "xMidYMid meet")
        .call(zoom)

      const link = d3.select('svg')
        .selectAll('line')
        .data(this.edges)
        .enter()
        .append('line')
        .attr('stroke-width', 1)
        .attr('stroke', 'LightGray')

      const nodeGroup = d3.select('svg')
        .selectAll('g')
        .data(this.nodes)
        .enter()
        .append('g')
        .call(d3.drag()
          .on('start', dragstarted)
          .on('drag', dragged)
          .on('end', dragended))
        .on('click', this.openPage)

      nodeGroup.append('ellipse')
        .attr('cx', d => d.x)
        .attr('cy', d => d.y)
        .attr('rx', d => d.rx)
        .attr('ry', d => d.ry)
        .attr('fill', d => d.user ? 'Cyan' : 'Gold')

      nodeGroup.append('text')
        .attr('x', d => d.x)
        .attr('y', d => d.y)
        .attr('text-anchor', 'middle')
        .attr('dominant-baseline', 'middle')
        .style('fill', 'steelbule')
        .style('font-size', '11px')
        .text(d => d.title)

      const simulation = d3.forceSimulation()
        .force('link',
          d3.forceLink()
            .distance(d => d.l)
            .iterations(2))
        .force('collide',
          d3.forceCollide()
            .radius(d => d.r)
            .strength(0.7)
            .iterations(2))
        .force('charge', d3.forceManyBody().strength(-100))
        .force('x', d3.forceX().strength(0.01).x(this.width / 2))
        .force('y', d3.forceY().strength(0.01).y(this.height / 2))
        .force('center', d3.forceCenter(this.width / 2, this.height / 2))

      simulation
        .nodes(this.nodes)
        .on('tick', ticked)

      simulation.force('link')
        .links(this.edges)
        .id(d => d.index)

      function ticked() {
        link
          .attr('x1', d => d.source.x)
          .attr('y1', d => d.source.y)
          .attr('x2', d => d.target.x)
          .attr('y2', d => d.target.y)
        nodeGroup.select('ellipse')
          .attr('cx', d => d.x)
          .attr('cy', d => d.y)
        nodeGroup.select('text')
          .attr('x', d => d.x)
          .attr('y', d => d.y)
      }

      function dragstarted(e, d) {
        if (!e.active) simulation.alphaTarget(0.3).restart()
        d.fx = d.x
        d.fy = d.y
      }

      function dragged(e, d) {
        d.fx = e.x
        d.fy = e.y
      }

      function dragended(e, d) {
        if (!e.active) simulation.alphaTarget(0)
        d.fx = null
        d.fy = null
      }

      link.call(zoom)
      nodeGroup.call(zoom)
    
      function zoomed(e, d) {
        link.attr('transform', e.transform)
        nodeGroup.attr('transform', e.transform)
      }
    },
    byteLength(str) {
      str = (str==null) ? "" : str
      return encodeURI(str).replace(/%../g, "*").length
    },
    openPage(e, d) {
      console.log(d)
      if (d.user) return
      const page = encodeURIComponent(d.title)
      const url = `https://scrapbox.io/${this.project}/${page}`
      window.open(url)
    }
  }
}
</script>

<style scoped>
  svg {
    background: white;
    position: fixed;
    top: 50px;
    left: 0;
    height: 100%;
    width: 100%
  }
</style>
